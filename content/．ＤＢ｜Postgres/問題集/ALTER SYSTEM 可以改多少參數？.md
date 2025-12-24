#postgreSQL 
PostgreSQL的`ALTER SYSTEM`只能修改**postgresql.conf**中的參數，不能修改其他設定檔案。

## ALTER SYSTEM的範圍

`ALTER SYSTEM`**只影響一個檔案**：

- ✅ **postgresql.conf** 中的參數
- ❌ **不能修改**：pg_hba.conf、pg_ident.conf等其他設定檔

## ALTER SYSTEM可修改的參數分類

根據資料中的資訊，PostgreSQL參數可以按照修改方式分為：

### 1. 立即生效的參數（不需重啟）

```sql
-- 記憶體相關
ALTER SYSTEM SET shared_buffers = '256MB';          -- 需重啟
ALTER SYSTEM SET work_mem = '4MB';                  -- 立即生效
ALTER SYSTEM SET maintenance_work_mem = '64MB';     -- 立即生效

-- 連線相關
ALTER SYSTEM SET max_connections = '200';           -- 需重啟
ALTER SYSTEM SET listen_addresses = '*';            -- 需重啟

-- WAL相關（多數需重啟）
ALTER SYSTEM SET wal_level = 'replica';             -- 需重啟
ALTER SYSTEM SET max_wal_size = '1GB';              -- 立即生效
ALTER SYSTEM SET wal_keep_size = '500MB';           -- 立即生效
```

### 2. 需要重啟的參數（postmaster parameters）

```sql
-- 這些修改後必須重啟
ALTER SYSTEM SET wal_level = 'replica';
ALTER SYSTEM SET max_connections = '100';
ALTER SYSTEM SET shared_buffers = '128MB';
ALTER SYSTEM SET listen_addresses = '*';
ALTER SYSTEM SET port = '5432';
```

## 查看參數可修改性

```sql
-- 查看所有參數的修改特性
SELECT name, context, pending_restart 
FROM pg_settings 
WHERE name LIKE '%wal%'
ORDER BY name;

-- context欄位說明：
-- 'internal'     - 編譯時設定，無法修改
-- 'postmaster'   - 需要重啟
-- 'sighup'       - 重新載入設定檔即可
-- 'superuser'    - superuser可隨時修改
-- 'user'         - 一般用戶可修改
```

## ALTER SYSTEM儲存位置

`ALTER SYSTEM`修改的參數會寫入：

```
postgresql.auto.conf
```

該檔案位於data目錄下，優先級高於postgresql.conf。

## 無法用ALTER SYSTEM修改的設定檔

### pg_hba.conf（認證設定）

```bash
# 必須手動編輯
host    all    all    192.168.1.0/24    md5
```

### pg_ident.conf（身份對應）

```bash
# 必須手動編輯
mapname    system-username    pg-username
```

### postgresql.conf中的某些特殊項目

```bash
# 這些通常需要手動編輯
include = 'other.conf'
include_dir = 'conf.d'
```

## 與Oracle的比較

**Oracle**：

- 可用`ALTER SYSTEM`修改spfile中的參數
- 需要重啟的參數會標示`SCOPE=SPFILE`
- 有些參數可以`SCOPE=BOTH`立即生效

**PostgreSQL**：

- `ALTER SYSTEM`只能修改postgresql.conf相關參數
- 參數按context分類決定是否需重啟
- 認證相關設定必須手動編輯其他設定檔

總的來說，PostgreSQL的`ALTER SYSTEM`功能比Oracle相對有限，主要專注在postgresql.conf的參數管理上。

# 複習與問題
1. [[WAL相關參數]]
2. [創建psql表空間的時候可以調整哪些參數？](創建psql表空間的時候可以調整哪些參數？.md)