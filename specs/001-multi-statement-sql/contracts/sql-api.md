# Contract: SQL 執行 API

兩個端點的請求和回應格式相同：

- `POST /api/sql`：MySQL
- `POST /api/pg/sql`：PostgreSQL

欄位定義見 [data-model.md](../data-model.md)。

## Request

```json
{
  "config": { "host": "10.89.0.3", "port": 3306, "user": "bigred", "password": "..." },
  "database": "test",
  "sql": "USE test;\nINSERT INTO accounts VALUES (1, 'Alice', 5000);\nSELECT * FROM accounts;"
}
```

`database` 可以不帶。

## Responses

### 200：全部成功

```json
{
  "results": [
    { "sql": "USE test", "type": "write", "affectedRows": 0, "insertId": null, "message": "執行成功" },
    { "sql": "INSERT INTO accounts VALUES (1, 'Alice', 5000)", "type": "write", "affectedRows": 1, "insertId": null, "message": "執行成功 — 影響 1 筆資料" },
    { "sql": "SELECT * FROM accounts", "type": "select", "rows": [{ "id": 1, "name": "Alice", "balance": 5000 }], "columns": ["id", "name", "balance"] }
  ]
}
```

- `results` 的順序跟 SQL 裡的段落順序相同。
- 只有一段時，`results` 長度是 1。

### 400：沒有可以執行的內容

```json
{ "error": "請輸入 SQL" }
```

```json
{ "error": "沒有可執行的 SQL" }
```

第一種是 `sql` 沒帶或是空字串。第二種是內容只有註解或空白。

### 500：執行失敗

某一段失敗：

```json
{
  "error": "第 3 段執行失敗：Duplicate entry '1' for key 'accounts.PRIMARY'",
  "failedIndex": 2,
  "failedSql": "INSERT INTO accounts VALUES (1,'dup',1)",
  "total": 4,
  "results": [
    { "sql": "USE test", "type": "write", "affectedRows": 0, "insertId": null, "message": "執行成功" },
    { "sql": "INSERT INTO accounts VALUES (3,'x',1)", "type": "write", "affectedRows": 1, "insertId": null, "message": "執行成功 — 影響 1 筆資料" }
  ]
}
```

只有一段而且失敗時，`error` 是資料庫原本的錯誤訊息，不加「第 1 段執行失敗：」，`total` 是 1，`results` 是空陣列。

連線失敗，或是 `database` 不存在導致 `USE` 失敗時，回應只有 `error`，沒有 `failedIndex` 這些欄位，跟其他 API 一樣：

```json
{ "error": "Unknown database 'nope'" }
```

## 相容性

這是不相容的變更。改版前成功時回傳 `{ type, rows, columns }` 或 `{ type, affectedRows, insertId, message }`，現在一律包在 `results` 陣列裡。目前只有 bobo 自己的前端呼叫這兩個端點，見 spec 的 Assumptions。
