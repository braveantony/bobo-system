# Research: SQL 分頁一次執行多段 SQL

這份記錄每個設計決定的理由，以及考慮過但沒採用的做法。R1 到 R6 是 commit c13d687 已經採用的做法。R7、R8 是補寫 spec 時才發現的落差，還沒實作。

## R1：怎麼把 SQL 切成多段

- Decision：用 `dbgate-query-splitter`，MySQL 用 `mysqlSplitterOptions`，PostgreSQL 用 `postgreSplitterOptions`。
- Rationale：它是 DbGate 這套網頁版資料庫工具在用的 splitter。實測過它能正確處理字串裡的分號（`'A;B'`）、`--` 和 `/* */` 註解、MySQL 的 `DELIMITER //`，以及 PostgreSQL 的 `$$` 函式本體，剛好涵蓋 FR-002 到 FR-004。套件本身很小，沒有原生模組，放進 `node:20-alpine` 不用額外處理。
- Alternatives considered：
  - 用 `;` 直接 split：字串和註解裡的分號會被切斷，不符合 FR-002。
  - 用 `node-sql-parser` 這類完整的 SQL parser：要能解析整句語法，遇到它不認得的語法就失敗，而且不支援 `DELIMITER`。
  - 自己寫 tokenizer：要處理的引號、跳脫字元和方言細節很多，維護成本高。

## R2：多段怎麼執行

- Decision：同一個 HTTP 請求裡開一條連線，依序一段一段執行，遇到錯誤就停。MySQL 維持 `multipleStatements: false`。
- Rationale：同一條連線才能讓 `USE`、`SET` 和交易對後面的段落生效（FR-005）。一段一段送，才知道是第幾段失敗、前面每段的結果是什麼（FR-009）。這也是 phpMyAdmin、DBeaver 這類工具的做法。
- Alternatives considered：
  - 開啟 `multipleStatements`，整串交給 MySQL：會放寬 SQL injection 一次塞多段的限制（違反 FR-016），出錯時也比較難對應到第幾段，PostgreSQL 那邊也要另外處理。
  - 前端切開，每段各送一個請求：每個請求都是新連線，`USE test` 對下一段沒有效果，不符合 FR-005。

## R3：怎麼判斷一段是查詢還是寫入

- Decision：看執行結果的形狀，不看 SQL 開頭的關鍵字。
  - MySQL：結果是陣列就是查詢。`CALL` 回傳的是「結果集陣列加上一個 header」，取第一個結果集。
  - PostgreSQL：結果有欄位（`fields` 不是空的）就是查詢，所以 `INSERT ... RETURNING` 也算查詢。
- Rationale：改版前是看開頭關鍵字，段落開頭有註解（例如 `-- 確認目前設定\nSELECT ...`）就會判斷錯。看結果形狀不受註解影響，`WITH`、`CALL`、`RETURNING` 也都不用另外列。
- Alternatives considered：沿用關鍵字白名單，再先去掉開頭註解。還是要一直補 `WITH`、`TABLE`、`VALUES` 這些關鍵字，容易漏。

## R4：只有註解的段落

- Decision：用 `sqlHead()` 把註解和空白拿掉，剩下空字串的段落就略過。`sqlHead()` 的結果只拿來判斷是否為空、以及判斷 `INSERT`、`UPDATE` 這類開頭，真正執行的還是原本的文字。
- Rationale：splitter 會把結尾的 `-- 備註` 也切成一段。這段送去 MySQL 會出現 `Query was empty` 錯誤。
- Alternatives considered：splitter 的 `ignoreComments` 選項。實測後它還是會留下只有空白和註解的段落，而且會把註解從要執行的 SQL 裡拿掉，執行紀錄就看不到使用者原本寫的內容。

## R5：失敗時怎麼回應

- Decision：回 HTTP 500，body 帶 `error`、`failedIndex`、`failedSql`、`total` 和已成功的 `results`。只有一段時，`error` 不加「第 1 段執行失敗：」。整段都是註解時回 400。
- Rationale：其他 API 遇到資料庫錯誤都是 500，維持一致。`total` 讓前端算得出後面還有幾段沒執行（FR-009），也能判斷是不是只有一段（FR-013）。
- Alternatives considered：回 200，在 body 裡標 `ok: false`。前端的 `api()` 是用 HTTP 狀態判斷成功或失敗，改成 200 要動到所有呼叫端。

## R6：API 回應格式

- Decision：成功時一律回 `{ results: [...] }`，一段 SQL 也是長度 1 的陣列。詳細格式在 [contracts/sql-api.md](./contracts/sql-api.md)。
- Rationale：一段和多段用同一種格式，前端只有一條處理路徑。照 spec 的 Assumptions，除了 bobo 自己的畫面沒有其他呼叫端，可以直接改格式。
- Alternatives considered：一段時維持舊格式 `{ type, rows, ... }`，多段才回陣列。前端要處理兩種格式，而且呼叫端事先不知道會切成幾段。

## R7：自動化測試（落差，還沒實作）

- Decision：用 Node.js 內建的 `node:test` 和 `node:assert`，分兩層：
  - `tests/sql-batch.test.js`：單元測試，測切分、空段落、訊息文字，不需要資料庫。
  - `tests/sql-api.test.js`：整合測試，啟動 `app` 後打 `/api/sql` 和 `/api/pg/sql`。用環境變數 `BOBO_TEST_MYSQL`、`BOBO_TEST_PG` 指定連線，沒有設定就略過該資料庫的測試。
  - `package.json` 加上 `"test": "node --test tests/"`。
- Rationale：專案目前沒有任何測試。commit c13d687 的驗證是用放在暫存目錄的腳本跑的，沒有留在 repo 裡，之後改程式沒辦法重跑。`node:test` 是 Node.js 20 內建的，不用多一個依賴。
- Alternatives considered：
  - Jest 或 Vitest：功能比較多，但為了十幾個測試多裝一整套框架不划算。
  - 把整合測試要用的資料庫寫成 docker-compose：比較完整，但 CI 還不存在，先用環境變數，等有 CI 再說。
- 為了讓測試能 `require` 程式，要做兩個小調整：多段 SQL 的邏輯搬到 `lib/sql-batch.js`；`server.js` 改成直接執行時才 `listen`，被 `require` 時只匯出 `app`。

## R8：執行紀錄被重新載入的資料表蓋掉（落差，待確認）

- 問題：左側已經選了某張資料表時，只要這次執行至少有一段成功、而且結果裡沒有查詢，前端會在 800ms 後呼叫 `fetchRows()` 重新載入那張表。`fetchRows()` 會把整個資料區換成表格，多段的執行紀錄和失敗資訊就不見了。這跟 FR-011（顯示每段紀錄）和 SC-003（只看畫面就知道失敗在哪）衝突，而且 FR-018 本身就要求重新載入，所以是 spec 裡兩條需求互相衝突。
- 為什麼之前沒發現：瀏覽器測試時左側沒有選資料表，不會觸發重新載入。
- 建議做法（待使用者確認）：畫面有顯示執行紀錄時，也就是多段執行或多段中途失敗時，不自動重新載入。只有一段 SQL 時維持改版前的行為，800ms 後重新載入。使用者要看最新資料，再點一次左側的資料表就好。FR-018 要改成只適用於一段 SQL 的情況。
- Alternatives considered：
  - 重新載入，但把表格畫在執行紀錄下方：`fetchRows()`、分頁和搜尋都要改成保留執行紀錄，改動比較大。
  - 維持現狀：違反 SC-003，失敗資訊只會閃一下就消失。
