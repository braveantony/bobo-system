# 🐡 bobo — MySQL & PostgreSQL Web 管理介面

輕量的資料庫管理工具，支援 MySQL 和 PostgreSQL。透過瀏覽器瀏覽與編輯資料、執行 SQL，也能管理資料庫、資料表、欄位和索引。

---

## 快速開始

### 1. 安裝依賴

```bash
npm install
```

### 2. 啟動伺服器

```bash
node server.js
```

或使用 nodemon 開發模式（自動重啟）：

```bash
npm run dev
```

### 3. 打開瀏覽器

```
http://localhost:3000
```

---

## 目錄結構

```
bobo/
├── server.js          # Express 後端 API
├── package.json
├── README.md
├── Dockerfile
├── docker-compose.yml
├── public/
│   └── index.html     # 前端 Web UI
└── yaml/              # Kubernetes 部署檔，見 yaml/infra/README.md
```

---

## API 說明

所有 API 都是 `POST`。MySQL 用 `/api/*`，PostgreSQL 用 `/api/pg/*`，兩組端點一一對應。

請求 body 都要帶 `config` 連線資訊。後端每個請求會用這份資訊開一條新連線，做完就關掉：

```json
{
  "config": {
    "host": "localhost",
    "port": 3306,
    "user": "root",
    "password": "yourpassword",
    "database": "yourdb",
    "ssl": false
  }
}
```

| 欄位 | MySQL | PostgreSQL |
| --- | --- | --- |
| `port` | 預設 3306 | 預設 5432 |
| `database` | 選填 | 要連的資料庫，沒填時後端用 `postgres`，前端則要求必填 |
| `ssl` | 不使用 | 選填，`true` 時用 SSL 連線，不驗證憑證 |

成功時回傳 JSON。失敗時回傳 HTTP 400（缺少參數）或 500（資料庫錯誤），格式是 `{"error": "錯誤訊息"}`。

### MySQL：瀏覽與資料操作

| 端點 | 功能 | 額外參數 | 回傳 |
| --- | --- | --- | --- |
| `/api/connect` | 測試連線 | — | `ok`, `version`, `user`, `now` |
| `/api/databases` | 列出資料庫，不含系統資料庫 | — | `databases` |
| `/api/tables` | 列出資料表 | `database` | `tables` |
| `/api/columns` | 取得欄位結構 | `database`, `table` | `columns` |
| `/api/rows` | 查詢資料（分頁） | `database`, `table`, `page`, `pageSize`, `search` | `rows`, `total`, `columns` |
| `/api/insert` | 新增一筆資料 | `database`, `table`, `data` | `ok`, `insertId` |
| `/api/update` | 更新資料 | `database`, `table`, `data`, `where` | `ok`, `affectedRows` |
| `/api/delete` | 刪除資料 | `database`, `table`, `where` | `ok`, `affectedRows` |
| `/api/sql` | 執行一段 SQL | `sql`，`database` 選填 | 見下方說明 |

- `page` 預設 1，`pageSize` 預設 50。`search` 會對所有欄位做 `LIKE '%關鍵字%'`。
- `data` 和 `where` 都是 `{欄位名: 值}` 的物件。`data` 裡的空字串會存成 `NULL`。`where` 的多個條件用 `AND` 串接，不可以是空物件。
- `/api/sql` 開頭是 `SELECT`、`SHOW`、`DESCRIBE`、`DESC`、`EXPLAIN` 時，回傳 `{type: "select", rows, columns}`。其他 SQL 也會執行，回傳 `{type: "write", affectedRows, insertId, message}`。一次只能執行一個 statement。

### MySQL：結構管理

| 端點 | 功能 | 額外參數 |
| --- | --- | --- |
| `/api/database/create` | 建立資料庫 | `database`，`charset` 預設 `utf8mb4`，`collation` 預設 `utf8mb4_unicode_ci` |
| `/api/database/drop` | 刪除資料庫 | `database` |
| `/api/table/create` | 建立資料表，回傳產生的 `sql` | `database`, `table`, `columns`，`engine` 預設 `InnoDB`，`charset` 預設 `utf8mb4` |
| `/api/table/drop` | 刪除資料表 | `database`, `table` |
| `/api/table/truncate` | 清空資料表 | `database`, `table` |
| `/api/table/rename` | 重新命名資料表 | `database`, `table`, `newName` |
| `/api/table/ddl` | 取得 `SHOW CREATE TABLE` 結果，回傳 `ddl` | `database`, `table` |
| `/api/column/add` | 新增欄位 | `database`, `table`, `column` |
| `/api/column/modify` | 修改欄位，可同時改名 | `database`, `table`, `oldName`, `column` |
| `/api/column/drop` | 刪除欄位 | `database`, `table`, `column`（欄位名稱字串） |
| `/api/indexes` | 列出索引，回傳 `indexes` | `database`, `table` |
| `/api/index/add` | 新增索引 | `database`, `table`, `indexName`, `columns`（欄位名稱陣列），`unique` 預設 `false` |
| `/api/index/drop` | 刪除索引 | `database`, `table`, `indexName` |

### PostgreSQL：瀏覽與資料操作

PG 的端點在 `/api/pg/` 底下，名稱和參數都跟 MySQL 那組一樣。差別在 `database` 參數：在 PG 這邊它指的是 schema，不是資料庫。要連哪個資料庫，由 `config.database` 決定。

| 端點 | 功能 | 跟 MySQL 的差異 |
| --- | --- | --- |
| `/api/pg/connect` | 測試連線 | `version` 只回版本號，例如 `16.2` |
| `/api/pg/databases` | 列出 schema | 排除 `information_schema` 和 `pg_*` |
| `/api/pg/tables` | 列出 schema 裡的資料表 | `CREATE_TIME` 固定是 `null`，`ENGINE` 固定是 `heap` |
| `/api/pg/columns` | 取得欄位結構 | 欄位名稱轉成跟 MySQL `SHOW FULL COLUMNS` 一樣的格式 |
| `/api/pg/rows` | 查詢資料（分頁） | `search` 用 `ILIKE`，不分大小寫 |
| `/api/pg/insert` | 新增一筆資料 | `insertId` 回傳的是整筆新資料（`RETURNING *`），不是單一 ID |
| `/api/pg/update` | 更新資料 | — |
| `/api/pg/delete` | 刪除資料 | — |
| `/api/pg/sql` | 執行一段 SQL | 有帶 `database` 時會先 `SET search_path TO <schema>, public`。開頭是 `SELECT`、`TABLE`、`EXPLAIN`、`WITH`、`SHOW` 時回傳 `select` 格式，`insertId` 固定是 `null` |

### PostgreSQL：結構管理

| 端點 | 功能 | 跟 MySQL 的差異 |
| --- | --- | --- |
| `/api/pg/database/create` | 建立 schema | 執行 `CREATE SCHEMA`，不支援 `charset`、`collation` |
| `/api/pg/database/drop` | 刪除 schema | 執行 `DROP SCHEMA ... CASCADE`，schema 裡的物件會一起刪掉 |
| `/api/pg/table/create` | 建立資料表，回傳產生的 `sql` | 不支援 `engine`、`charset` |
| `/api/pg/table/drop` | 刪除資料表 | — |
| `/api/pg/table/truncate` | 清空資料表 | — |
| `/api/pg/table/rename` | 重新命名資料表 | — |
| `/api/pg/table/ddl` | 取得建表語法，回傳 `ddl` | 由 `information_schema` 組出來，只包含欄位、`NOT NULL`、預設值和主鍵 |
| `/api/pg/column/add` | 新增欄位 | — |
| `/api/pg/column/modify` | 修改欄位 | 分成好幾個 `ALTER` 依序執行：改名、`TYPE ... USING`、`NOT NULL`、`DEFAULT` |
| `/api/pg/column/drop` | 刪除欄位 | — |
| `/api/pg/indexes` | 列出索引 | 欄位名稱轉成跟 MySQL `SHOW INDEX` 一樣的格式，`Cardinality` 放的是 `idx_scan` |
| `/api/pg/index/add` | 新增索引 | 執行 `CREATE INDEX ... ON` |
| `/api/pg/index/drop` | 刪除索引 | — |

### 欄位定義物件

`table/create` 的 `columns` 陣列，以及 `column/add`、`column/modify` 的 `column`，都用這個格式：

```json
{
  "name": "id",
  "type": "INT",
  "length": 11,
  "unsigned": true,
  "notNull": true,
  "default": "",
  "autoIncrement": true,
  "comment": "主鍵",
  "primaryKey": true,
  "unique": false,
  "index": false
}
```

| 屬性 | 說明 |
| --- | --- |
| `name`, `type` | 必填 |
| `length` | 長度。PG 只在 `VARCHAR`、`CHAR`、`DECIMAL`、`NUMERIC` 上生效 |
| `notNull` | 加上 `NOT NULL` |
| `default` | 預設值。`NULL` 和 `CURRENT_TIMESTAMP` 照字面使用，其他值當字串處理 |
| `autoIncrement` | MySQL 加上 `AUTO_INCREMENT`。PG 會把型別換成 `SERIAL`、`BIGSERIAL` 或 `SMALLSERIAL` |
| `unsigned`, `comment` | 只有 MySQL 支援 |
| `primaryKey`, `unique` | 只在 `table/create` 生效 |
| `index` | 只在 MySQL 的 `table/create` 生效，會建立 `idx_<欄位名>` 索引 |
| `after`, `first` | 只在 MySQL 的 `column/add` 生效，決定新欄位的位置 |

---

## 安全性說明

- MySQL 的識別符（資料庫名、資料表名、欄位名）會經過白名單過濾，只保留英數字、`_`、`-`、`.` 和空白。PG 的識別符用雙引號包起來並跳脫。
- 資料值都使用參數化查詢。
- `/api/sql` 和 `/api/pg/sql` 會執行任何 SQL，包含寫入和 DDL。只看 SQL 開頭決定回傳格式，不會擋寫入操作。
- 每次請求獨立建立連線，不共用 connection pool。
- 連線帳號密碼每次請求都放在 body 裡送出，後端也開了 CORS。請不要把 bobo 開放在不受信任的網路上。

---

## 修改 Port

```bash
PORT=8080 node server.js
```

或直接修改 `server.js` 第 11 行：

```js
const PORT = process.env.PORT || 3000;
```
