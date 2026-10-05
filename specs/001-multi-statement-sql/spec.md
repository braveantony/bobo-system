# Feature Specification: SQL 分頁一次執行多段 SQL

**Feature Branch**: `worktree-sql-multi-statement`

**Created**: 2026-10-04

**Status**: Draft

**Input**: 使用者描述："SQL 分頁支援一次執行多段 SQL。使用者在 bobo 的 SQL 編輯框貼上用分號分隔的多段 SQL（例如 USE test; CREATE TABLE ...; INSERT ...; SELECT ...），按執行後，後端用懂 SQL 語法的 splitter 切開，在同一條資料庫連線依序執行，讓 USE、SET 和同一批裡的交易有效。MySQL 和 PostgreSQL 都要支援。字串和註解裡的分號不能被切開，MySQL 要支援 DELIMITER，PG 要支援 $$ 函式本體。遇到錯誤就停，回報失敗的是第幾段、已經成功的段落和還沒執行的段數。畫面要顯示每一段的執行結果，以及最後一個查詢的結果表格。MySQL 的 multipleStatements 要維持關閉。跨多次執行保留連線（互動式交易）不在範圍內。"

這份 spec 是功能做完之後補寫的。實作在 commit c13d687，內容依照當時的討論和實作整理，還沒經過使用者逐條確認。

## Clarifications

### Session 2026-10-04

- Q: 多段 SQL 中途出錯時，前面已經成功的段落要保留，還是整批自動復原？ → A: 保留已成功的段落，不自動復原。要全有或全無，使用者自己寫 `START TRANSACTION ... COMMIT`。

### Session 2026-10-05

- Q: 一次執行裡有好幾個查詢時，畫面要怎麼顯示這些查詢結果？ → A: 表格只顯示最後一個查詢的結果，前面的查詢在執行紀錄裡只顯示筆數。

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 貼上一整段 SQL 直接執行 (Priority: P1)

使用者從教材或筆記複製一段 SQL，裡面有好幾個用分號隔開的語句，例如先 `USE test`，再建表、新增資料，最後查詢設定值。使用者把整段貼進 SQL 編輯框，按一次執行，所有語句就依序跑完。畫面列出每一段的執行結果，下面顯示最後一個查詢的結果表格。

改版前，這種 SQL 會直接出現語法錯誤，使用者只能一段一段拆開貼。

**Why this priority**: 這是使用者回報的問題本身。做完這個，最常見的「貼一段練習用 SQL 就跑」就能用了。

**Independent Test**: 連上 MySQL，貼上四段 SQL（`USE`、`CREATE TABLE`、`INSERT` 兩筆、`SELECT @@autocommit`），按一次執行，確認四段都成功，表也確實建好、資料有兩筆。

**Acceptance Scenarios**:

1. **Given** 已連上 MySQL，**When** 使用者貼上 `USE test; CREATE TABLE accounts (...); INSERT INTO accounts VALUES (1,'Alice',5000),(2,'Bob',3000); SELECT @@autocommit, @@transaction_isolation, @@version;` 並按執行，**Then** 畫面列出 4 段紀錄，每段前面打勾，第 3 段顯示「執行成功 — 影響 2 筆資料」，下方表格顯示第 4 段的查詢結果，上方提示「第 4 段的查詢結果 — 1 筆」。
2. **Given** 已連上 PostgreSQL 並選了某個 schema，**When** 使用者貼上 `CREATE TABLE ...; INSERT ...; SELECT ...;` 並按執行，**Then** 三段都在選定的 schema 裡執行成功，畫面顯示最後一段的查詢結果。
3. **Given** SQL 的字串裡有分號，例如 `INSERT INTO t VALUES ('A;B'); SELECT * FROM t;`，**When** 使用者按執行，**Then** 系統把它當成 2 段，存進去的值是 `A;B`。
4. **Given** 每一段前面都有 `--` 註解，**When** 使用者按執行，**Then** 註解不會讓該段失敗，執行紀錄顯示的是去掉開頭註解後的 SQL。
5. **Given** 編輯框只有一段 SQL，**When** 使用者按執行，**Then** 畫面跟改版前一樣：查詢顯示結果表格，寫入顯示「執行成功」和影響筆數，不顯示執行紀錄。

---

### User Story 2 - 中途出錯時知道停在哪裡 (Priority: P2)

使用者執行一段多段 SQL，其中某一段出錯，例如主鍵重複。系統停在出錯的那段，後面的不執行。畫面上清楚標出哪幾段已經成功、哪一段失敗、錯誤訊息是什麼、後面還有幾段沒有執行，讓使用者知道資料庫現在是什麼狀態。

**Why this priority**: 多段執行最怕的是出錯後不知道做到哪裡。沒有這個，使用者可能重跑整段，造成重複寫入。

**Independent Test**: 執行 `INSERT id=4; INSERT id=1（主鍵重複）; INSERT id=5`，確認畫面標出第 2 段失敗、後面 1 段沒執行，再查表確認 id=4 存在、id=5 不存在。

**Acceptance Scenarios**:

1. **Given** 表裡已經有 id=1，**When** 使用者執行 `USE test; INSERT ... id=3; INSERT ... id=1; SELECT 1`，**Then** 第 1、2 段打勾，第 3 段用紅色標示並顯示資料庫的錯誤訊息，下面提示「後面 1 段沒有執行。前面已成功的段落不會自動復原，除非它們在還沒 COMMIT 的交易裡。」，右下角提示「SQL 錯誤：第 3 段執行失敗：……」。
2. **Given** 同上，**When** 執行結束後使用者查詢該表，**Then** id=3 已經存在，第 4 段沒有被執行。
3. **Given** 只有一段 SQL 而且有語法錯誤，**When** 使用者按執行，**Then** 只跳出錯誤提示，訊息不加「第 1 段執行失敗」，也不顯示執行紀錄。
4. **Given** 第 1 段就失敗，**When** 使用者按執行，**Then** 提示只寫後面幾段沒有執行，不顯示「前面已成功的段落不會自動復原」。

---

### User Story 3 - 同一次執行裡的 session 狀態和交易有效 (Priority: P3)

使用者在同一段 SQL 裡用 `USE` 切換資料庫、用 `SET` 設定 session 變數、用 `START TRANSACTION` 開交易再 `COMMIT` 或 `ROLLBACK`。這些設定在同一次執行的後續語句都有效。

**Why this priority**: 很多練習用 SQL 會先 `USE` 或 `SET`。交易也是常見的練習主題，至少要能在一次執行裡完整跑完一個交易。

**Independent Test**: 執行 `START TRANSACTION; UPDATE ... balance-1000; SELECT balance; ROLLBACK; SELECT balance`，確認第 3 段看到扣款後的值，第 5 段看到原值。

**Acceptance Scenarios**:

1. **Given** id=1 的 balance 是 5000，**When** 使用者執行 `START TRANSACTION; UPDATE accounts SET balance=balance-1000 WHERE id=1; SELECT balance FROM accounts WHERE id=1; ROLLBACK; SELECT balance FROM accounts WHERE id=1`，**Then** 第 3 段的結果是 4000，第 5 段的結果是 5000。
2. **Given** 已連上 MySQL，**When** 使用者執行 `SET @x = 41; SELECT @x + 1 AS v`，**Then** 結果是 42。
3. **Given** 左側選的是資料庫 A，**When** 使用者執行 `USE B; SELECT DATABASE()`，**Then** 結果是 B。
4. **Given** PostgreSQL 上執行 `BEGIN; INSERT ...; SELECT 不存在的欄位; COMMIT`，**When** 第 3 段失敗，**Then** 第 2 段新增的資料不會留在資料庫裡。

---

### User Story 4 - 建立 stored procedure 和函式 (Priority: P4)

使用者要建立 MySQL 的 stored procedure 或 PostgreSQL 的函式。這些語法的本體裡本來就有分號，系統要能正確辨認語句的邊界，不能從本體中間切斷。

**Why this priority**: 用到的人比較少，但切錯會讓建立失敗，而且錯誤訊息很難看懂。

**Independent Test**: 在 MySQL 執行 `DELIMITER //` 包起來的 `CREATE PROCEDURE` 加上 `CALL`，在 PG 執行 `$$` 包起來的 `CREATE FUNCTION` 加上 `SELECT f()`，兩者都要成功並回傳結果。

**Acceptance Scenarios**:

1. **Given** 已連上 MySQL，**When** 使用者執行 `DELIMITER //` 換行 `CREATE PROCEDURE p() BEGIN SELECT 7 AS n; END//` 換行 `DELIMITER ;` 換行 `CALL p();`，**Then** 系統把它當成 2 段，第 2 段顯示查詢結果 n=7。
2. **Given** 已連上 PostgreSQL，**When** 使用者執行 `CREATE FUNCTION f() RETURNS int AS $$ BEGIN RETURN 1; END $$ LANGUAGE plpgsql; SELECT f() AS v`，**Then** 系統把它當成 2 段，第 2 段的結果是 v=1。

---

### Edge Cases

- 編輯框裡只有註解：不執行任何東西，提示「沒有可執行的 SQL」。
- 最後一段後面只剩註解，例如結尾多一行 `-- 備註`：那段註解直接略過，不算一段，也不會出錯。
- 連續兩個分號 `;;`，中間是空的：空的段落直接略過。
- 多段裡有好幾個查詢：表格只顯示最後一個查詢的結果，前面的查詢在執行紀錄裡只顯示筆數。
- 查詢回傳 0 筆：算查詢結果，表格只有欄位名稱，紀錄顯示「0 筆」。
- PostgreSQL 的 `INSERT ... RETURNING`：有回傳欄位，當成查詢結果顯示。
- MySQL 的 `CALL` 回傳多個結果集：只顯示第一個結果集。
- SQL 或錯誤訊息裡有 `<b>` 這類 HTML：畫面照字面顯示，不會被當成 HTML。
- 中途失敗、但前面有查詢成功：表格仍然顯示最後一個成功的查詢結果，上方同時列出執行紀錄和失敗的段落。
- 下一次按執行：是全新的連線，上一次的 `USE`、session 變數和還沒 `COMMIT` 的交易都不存在。
- MySQL 交易裡遇到 DDL：`CREATE`、`DROP`、`ALTER`、`TRUNCATE`、`RENAME` 這類語句會讓 MySQL 自動 COMMIT 目前的交易。例如執行 `START TRANSACTION; INSERT ...; CREATE TABLE ...; 出錯的語句`，INSERT 在 `CREATE TABLE` 執行時就已經寫進資料庫，後面出錯也不會被 rollback。PostgreSQL 的 DDL 可以放在交易裡，沒有這個問題。

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系統 MUST 把 SQL 編輯框的內容依分號切成多段，依照原本的順序一段一段執行。
- **FR-002**: 系統 MUST NOT 切開單引號、雙引號、反引號字串裡的分號，也 MUST NOT 切開 `--`、`#`、`/* */` 註解裡的分號。
- **FR-003**: 連 MySQL 時，系統 MUST 支援 `DELIMITER` 指令：換了分隔符號之後，用新的符號判斷語句結尾，`DELIMITER` 這一行本身不送去資料庫執行。
- **FR-004**: 連 PostgreSQL 時，系統 MUST NOT 切開 `$$` 或 `$tag$` 包起來的內容。
- **FR-005**: 同一次執行的所有段落 MUST 在同一個資料庫 session 裡執行，讓 `USE`、`SET`、`SET search_path`、`START TRANSACTION`、`BEGIN`、`COMMIT`、`ROLLBACK` 對後面的段落生效。
- **FR-006**: 拿掉註解和空白後沒有內容的段落 MUST 略過，不送去執行，也不算進段數。
- **FR-007**: 整個編輯框都沒有可執行的段落時，系統 MUST 不執行任何東西，並提示「沒有可執行的 SQL」。
- **FR-008**: 某一段執行失敗時，系統 MUST 停止，後面的段落 MUST NOT 執行。
- **FR-009**: 失敗時，系統 MUST 回報失敗的是第幾段、那一段的 SQL、資料庫的錯誤訊息、已經成功的段落結果，以及後面還有幾段沒有執行。
- **FR-010**: 失敗時，系統 MUST NOT 自動復原已經成功的段落，也 MUST NOT 自動把整批包在交易裡。使用者自己開的交易如果還沒 `COMMIT`，會在這次執行結束、session 關閉時被資料庫 rollback。
- **FR-011**: 段數超過 1 段時，畫面 MUST 列出每一段的執行紀錄，包含段落編號、成功或失敗、去掉開頭註解後的 SQL，以及結果。查詢的結果寫「N 筆」。`INSERT`、`UPDATE`、`DELETE`、`REPLACE` 寫「執行成功 — 影響 N 筆資料」。其他語句寫「執行成功」。
- **FR-012**: 有任何一段回傳查詢結果時，畫面 MUST 只用一張表格顯示最後一個查詢結果，前面的查詢只在執行紀錄裡顯示筆數。多段時，表格上方 MUST 標示「第 N 段的查詢結果 — M 筆」。
- **FR-013**: 只有一段 SQL 時，畫面和提示 MUST 跟改版前一樣：不顯示執行紀錄，錯誤訊息不加「第 1 段執行失敗」。
- **FR-014**: 多段全部成功時，系統 MUST 提示「N 段 SQL 全部執行成功」。多段中途失敗時，提示 MUST 是「SQL 錯誤：第 N 段執行失敗：<資料庫錯誤訊息>」。
- **FR-015**: 畫面顯示 SQL 文字和錯誤訊息時，MUST 照字面顯示，不能被瀏覽器當成 HTML 解讀。
- **FR-016**: 系統每次只能把一段 SQL 交給資料庫。MySQL 連線 MUST NOT 開啟「一次送多段 SQL」的模式。
- **FR-017**: 每次按執行 MUST 用新的 session。上一次執行留下的 session 狀態 MUST NOT 帶到下一次。
- **FR-018**: 執行結束後，如果有任何寫入成功，而且左側目前選著某張資料表，系統 MUST 重新載入那張表的資料，跟改版前一致。

### Key Entities *(include if feature involves data)*

- **執行批次**：使用者按一次執行送出的整段 SQL。一個批次對應一個資料庫 session，裡面有一到多個段落。
- **段落**：批次切開後的一個 SQL 語句，有自己的編號（從 1 開始）和原始文字，原始文字包含開頭的註解。
- **段落結果**：一個段落執行後的結果。分成兩種：查詢結果有欄位名稱和資料列；寫入結果有影響筆數、新增的 ID（如果有的話）和一句訊息。
- **失敗資訊**：批次中途失敗時產生，包含失敗段落的編號和 SQL、錯誤訊息、總段數，以及失敗前已成功的段落結果。

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 使用者回報出錯的那段四語句 SQL，在 MySQL 上按一次執行就全部成功，不需要拆開。
- **SC-002**: 一組包含字串裡的分號、註解裡的分號、MySQL `DELIMITER`、PostgreSQL `$$` 函式的測試 SQL，切出來的段數和每段內容 100% 符合預期。
- **SC-003**: 中途失敗時，使用者只看畫面（不看伺服器 log）就能說出失敗的是第幾段、哪幾段已經執行、後面還有幾段沒有執行。
- **SC-004**: 只執行一段 SQL 時，查詢結果、寫入訊息和錯誤提示跟改版前相同。
- **SC-005**: 同一次執行裡的交易測試（扣款、查詢、ROLLBACK、再查詢）在 MySQL 和 PostgreSQL 上都得到「扣款後的值、原值」這組結果。

## Assumptions

- 一個批次沒有段數上限，每段的逾時沿用原本的連線設定。
- MySQL 的新增 ID 在資料表沒有 auto increment 時顯示為沒有值，不顯示 0。
- 跨多次執行保留 session（互動式交易）不在這次範圍內，要做交易練習的人用 mysql 或 psql CLI。
- 除了 bobo 自己的畫面，沒有其他程式依賴 SQL 執行結果的舊格式，所以結果格式可以跟著這次改動調整。
- 這個功能已經實作在 commit c13d687，這份 spec 是事後補寫的，後續會用 `/speckit-clarify` 確認上面的假設，再用 `/speckit-converge` 比對程式和 spec 的差異。
