# Quickstart: 驗證多段 SQL 執行

照這份步驟可以從頭驗證這個功能。API 格式見 [contracts/sql-api.md](./contracts/sql-api.md)，畫面行為見 [contracts/sql-tab-ui.md](./contracts/sql-tab-ui.md)。

## 事前準備

- Node.js 20 以上
- 一個可以連的 MySQL，帳號要能建立和刪除資料庫。下面用 cluster 裡的 `10.89.0.3:3306`，帳號 `bigred`
- 一個可以連的 PostgreSQL。沒有的話用 podman 起一個暫時的：

```sh
podman run -d --rm --name bobo-pgtest -e POSTGRES_PASSWORD=pgtest \
  -p 127.0.0.1:55432:5432 docker.io/library/postgres:16-alpine
```

## 啟動

```sh
npm install
PORT=3901 node server.js
```

下面的範例都打 `http://127.0.0.1:3901`。測試用的資料都放在 `bobo_dev_test` 資料庫或 schema，最後會刪掉。

為了少打字，先設好兩個 shell 函式：

```sh
# 用 node 組 JSON，避免 SQL 裡的引號和換行要手動跳脫
body() { node -e 'const [cfg,sql,db]=process.argv.slice(1);
  process.stdout.write(JSON.stringify({config:JSON.parse(cfg),sql,...(db?{database:db}:{})}))' "$@"; }
call() { curl -s -w ' [HTTP %{http_code}]\n' -X POST "http://127.0.0.1:3901$1" \
  -H 'Content-Type: application/json' -d @-; }
my() { body '{"host":"10.89.0.3","user":"bigred","password":"bigred"}' "$1" "${2:-}" | call /api/sql; }
pg() { body '{"host":"127.0.0.1","port":55432,"user":"postgres","password":"pgtest","database":"postgres"}' "$1" "${2:-}" | call /api/pg/sql; }
```

每個指令最後會印出 HTTP 狀態碼，例如 `[HTTP 200]`。

## 情境

### 1. 貼上一整段 SQL（User Story 1）

```sh
my "CREATE DATABASE bobo_dev_test;
USE bobo_dev_test;
-- 必須是 InnoDB
CREATE TABLE accounts (id INT PRIMARY KEY, name VARCHAR(20), balance INT NOT NULL) ENGINE=InnoDB;
INSERT INTO accounts VALUES (1, 'Alice', 5000), (2, 'Bob', 3000);
-- 確認目前設定
SELECT @@autocommit, @@transaction_isolation, @@version;"
```

預期：HTTP 200，`results` 有 5 筆。第 4 筆的 `message` 是「執行成功 — 影響 2 筆資料」，第 5 筆的 `type` 是 `select`。

字串裡的分號：

```sh
my "INSERT INTO accounts VALUES (3, 'C;D -- x', 1); SELECT name FROM accounts WHERE id=3" bobo_dev_test
```

預期：2 筆結果，第 2 筆的 `rows[0].name` 是 `C;D -- x`。

### 2. 中途出錯（User Story 2）

```sh
my "INSERT INTO accounts VALUES (4,'E',1); INSERT INTO accounts VALUES (1,'dup',1); INSERT INTO accounts VALUES (5,'F',1)" bobo_dev_test
my "SELECT id FROM accounts WHERE id IN (4,5)" bobo_dev_test
```

預期：第一個指令回 HTTP 500，`failedIndex` 是 1，`total` 是 3，`results` 有 1 筆，`error` 開頭是「第 2 段執行失敗：」。第二個指令只查到 id=4。

只有一段而且失敗：

```sh
my "SELEC 1" bobo_dev_test
```

預期：HTTP 500，`error` 開頭是「You have an error in your SQL syntax」，沒有「第 1 段」，`total` 是 1。

只有註解：

```sh
my "-- 只有註解" bobo_dev_test
```

預期：HTTP 400，`error` 是「沒有可執行的 SQL」。

### 3. session 狀態和交易（User Story 3）

```sh
my "START TRANSACTION; UPDATE accounts SET balance=balance-1000 WHERE id=1; SELECT balance FROM accounts WHERE id=1; ROLLBACK; SELECT balance FROM accounts WHERE id=1" bobo_dev_test
my "SET @x = 41; SELECT @x + 1 AS v"
```

預期：第一個指令第 3 筆的 balance 是 4000，第 5 筆是 5000。第二個指令的 v 是 42。

### 4. procedure 和函式（User Story 4）

```sh
my "DELIMITER //
CREATE PROCEDURE p() BEGIN SELECT 7 AS n; END//
DELIMITER ;
CALL p();" bobo_dev_test
pg "CREATE SCHEMA bobo_dev_test"
pg "CREATE FUNCTION f() RETURNS int AS \$\$ BEGIN RETURN 1; END \$\$ LANGUAGE plpgsql; SELECT f() AS v" bobo_dev_test
```

預期：MySQL 回 2 筆結果，第 2 筆 n=7。PostgreSQL 回 2 筆結果，第 2 筆 v=1。

PostgreSQL 交易失敗時 rollback：

```sh
pg "CREATE TABLE t (id INT); BEGIN; INSERT INTO t VALUES (9); SELECT nope; COMMIT" bobo_dev_test
pg "SELECT count(*) AS n FROM t" bobo_dev_test
```

預期：第一個指令在第 4 段失敗。第二個指令的 n 是 `"0"`。

### 5. 畫面

在瀏覽器開 `http://127.0.0.1:3901`，連上 MySQL，依照 [contracts/sql-tab-ui.md](./contracts/sql-tab-ui.md) 的表格逐列確認。至少要測：

- 情境 1 的 SQL：上方有 5 行執行紀錄，下方是 `@@autocommit` 那張表，提示列寫「第 5 段的查詢結果 — 1 筆」。
- 情境 2 的 SQL：第 2 行標紅，下面有「後面 1 段沒有執行。」的說明。
- 先點左側某張資料表，再執行情境 2 的 SQL，確認執行紀錄沒有被蓋掉（research.md R8，要等這個落差修好才會通過）。
- 在 SQL 裡放 `'<b>x</b>'`，確認紀錄照字面顯示。

## 自動化測試

做完 research.md R7 之後：

```sh
npm test
BOBO_TEST_MYSQL='{"host":"10.89.0.3","user":"bigred","password":"bigred"}' \
BOBO_TEST_PG='{"host":"127.0.0.1","port":55432,"user":"postgres","password":"pgtest","database":"postgres"}' \
npm test
```

第一個只跑單元測試，第二個會連資料庫跑整合測試。

## 清理

```sh
my "DROP DATABASE bobo_dev_test"
pg "DROP SCHEMA bobo_dev_test CASCADE"
podman stop bobo-pgtest
```
