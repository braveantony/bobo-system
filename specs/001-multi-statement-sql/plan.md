# Implementation Plan: SQL 分頁一次執行多段 SQL

**Branch**: `worktree-sql-multi-statement` | **Date**: 2026-10-05 | **Spec**: [spec.md](./spec.md)

**Input**: 功能規格，位於 `specs/001-multi-statement-sql/spec.md`

**Note**: 這份 plan 是功能做完之後補寫的。實作在 commit c13d687，這裡記錄的是實際採用的設計，以及對照 spec 後發現的落差。

## Summary

使用者在 SQL 分頁貼上多段 SQL 時，原本會出現語法錯誤，因為前端把整段送出，後端的 MySQL 連線又關掉了一次送多段的模式。

做法是在後端用 `dbgate-query-splitter` 把 SQL 切成一段一段，在同一條連線上依序執行，遇到錯誤就停。API 回應改成每一段一筆結果的陣列。前端依照結果數量決定畫面：一段時跟改版前一樣，多段時顯示執行紀錄和最後一個查詢結果。MySQL 的 `multipleStatements` 維持關閉。

對照 spec 後有兩個落差要處理，細節在 [research.md](./research.md) 的 R7、R8：

- 已經選了資料表時，執行紀錄在 800ms 後會被重新載入的資料表蓋掉，跟 FR-011 衝突。處理方式待確認。
- 專案目前沒有自動化測試，驗收要靠手動。這次補上 `node:test` 的測試。

## Technical Context

**Language/Version**: Node.js 20（Docker image 用 `node:20-alpine`），本機開發用 Node.js 22 也可以跑

**Primary Dependencies**: Express 4、mysql2 3、pg 8、cors，以及這次新增的 dbgate-query-splitter 4.12

**Storage**: bobo 自己不存資料。連到使用者指定的 MySQL 或 PostgreSQL，連線資訊每次請求由前端帶過來

**Testing**: 目前沒有任何測試。這次採用 Node.js 內建的 `node:test`，不另外裝測試框架，見 research.md R7

**Target Platform**: Linux。用 Docker image 部署到 kind 上的 Kubernetes，replicas 1

**Project Type**: web-service，後端一支 `server.js`，前端是單一檔案 `public/index.html`，用原生 JavaScript，沒有 build 流程

**Performance Goals**: 沒有特別的目標。照 FR-019，不限制段數和總執行時間

**Constraints**: MySQL 連線 `multipleStatements: false`（FR-016）。每個 HTTP 請求開一條新連線，結束就關（FR-017）。前端維持單一檔案、不引入 build 工具

**Scale/Scope**: 目前沒有登入，單一 bobo 實例。之後會做多使用者登入，到時要重新檢討 FR-019

## Constitution Check

*GATE：Phase 0 研究開始前要先通過，Phase 1 設計完成後再檢查一次。*

constitution 1.0.0 的三條原則都是針對 spec-kit 產出文件的寫法，不限制程式設計。

| 原則 | 檢查內容 | Phase 0 前 | Phase 1 後 |
| --- | --- | --- | --- |
| I. Taiwan Traditional Chinese | plan、research、data-model、contracts、quickstart 都用台灣繁體中文，沒有中國大陸用詞 | 通過 | 通過 |
| II. Human Voice | 沒有 AGENT.md 列的套話，一句講一件事 | 通過 | 通過 |
| III. Concrete and Testable | 設計決定都有具體行為，不確定的地方標「待確認」 | 通過 | 通過，R8 標為待確認 |

沒有違反的地方，Complexity Tracking 不用填。

## Project Structure

### Documentation (this feature)

```text
specs/001-multi-statement-sql/
├── plan.md              # 這份檔案
├── research.md          # 設計決定和落差（Phase 0）
├── data-model.md        # 執行批次、段落、結果的結構（Phase 1）
├── quickstart.md        # 手動和自動驗證步驟（Phase 1）
├── contracts/
│   ├── sql-api.md       # /api/sql、/api/pg/sql 的請求和回應
│   └── sql-tab-ui.md    # SQL 分頁各種狀態下的畫面
├── checklists/
│   └── requirements.md  # spec 品質檢查清單
└── tasks.md             # /speckit-tasks 產出
```

### Source Code (repository root)

```text
server.js                # Express 後端。/api/sql、/api/pg/sql 改成多段執行
lib/
└── sql-batch.js         # 新增：從 server.js 搬出來的切分和逐段執行邏輯，方便單元測試
public/
└── index.html           # SQL 分頁的執行紀錄、結果顯示、錯誤顯示
tests/
├── sql-batch.test.js    # 新增：切分、空段落、訊息文字的單元測試，不需要資料庫
└── sql-api.test.js      # 新增：打 /api/sql、/api/pg/sql 的整合測試，需要 MySQL 和 PostgreSQL
package.json             # 新增 dbgate-query-splitter 依賴和 test script
README.md                # 「執行 SQL」一節說明新的行為和回應格式
```

**Structure Decision**: 維持現有的單一專案結構，不拆 backend 和 frontend 目錄。唯一的結構調整是把多段 SQL 的邏輯從 `server.js` 搬到 `lib/sql-batch.js`，讓單元測試可以直接 `require`，不用啟動整個伺服器。`server.js` 改成被 `require` 時只匯出 `app`、直接執行時才 `listen`，讓整合測試可以自己決定 port。

## Complexity Tracking

Constitution Check 沒有違反的地方，不需要填寫。
