#postgreSQL 
## 基本概念差異

根據教材：「**pg_dump：用於備份單一資料庫**」「**pg_restore：從 pg_dump 創建的備份文件中恢復資料**」

|工具|作用|方向|
|---|---|---|
|**pg_dump**|備份/匯出|從資料庫 → 檔案|
|**pg_restore**|還原/匯入|從檔案 → 資料庫|

## 詳細差異

### pg_dump - 匯出工具

**用途：** 將資料庫內容匯出成檔案

```bash
# 基本語法
pg_dump [選項] 資料庫名稱 > 檔案名稱

# 實際範例
pg_dump -U postgres company_db > backup.sql
```

**輸出格式：**

- **預設：純文字SQL檔案** (`.sql`)
- **自定義格式：** 使用 `-Fc` 參數 (`.dump`)
- **目錄格式：** 使用 `-Fd` 參數

### pg_restore - 匯入工具

**用途：** 將pg_dump產生的檔案還原到資料庫

**重要限制：** `pg_restore` **只能處理非純文字格式**的檔案！

```bash
# 基本語法
pg_restore [選項] -d 目標資料庫 備份檔案

# 實際範例
pg_restore -U postgres -d company_db backup.dump
```

## 關鍵差異：檔案格式處理

### 情況1：純文字SQL檔案 (.sql)

```bash
# 匯出
pg_dump -U postgres company_db > backup.sql

# 匯入 - 只能用 psql，不能用 pg_restore
psql -U postgres -d company_db -f backup.sql
```

### 情況2：自定義格式檔案 (.dump)

```bash
# 匯出
pg_dump -U postgres -Fc company_db > backup.dump

# 匯入 - 可以用 pg_restore
pg_restore -U postgres -d company_db backup.dump
```

## 比較表

| 特性         | pg_dump | pg_restore |
| ---------- | ------- | ---------- |
| **主要用途**   | 備份/匯出   | 還原/匯入      |
| **支援格式**   | 可產生多種格式 | 只支援二進位格式   |
| **純文字SQL** | 可產生     | **不支援**    |
| **壓縮**     | 可選      | 自動處理       |
| **平行處理**   | 部分支援    | 完整支援       |
| **選擇性還原**  | N/A     | 支援         |

## 實務應用場景

### 使用 pg_dump + psql（純文字方式）

```bash
# 適用：小型資料庫、易讀格式、跨版本相容
pg_dump -U postgres company_db > backup.sql
psql -U postgres -d new_company_db -f backup.sql
```

### 使用 pg_dump + pg_restore（二進位方式）

```bash
# 適用：大型資料庫、需要壓縮、選擇性還原
pg_dump -U postgres -Fc company_db > backup.dump
pg_restore -U postgres -d new_company_db backup.dump

# 選擇性還原（只還原特定表格）
pg_restore -U postgres -d new_company_db -t employees backup.dump
```

## 與Oracle的類比

如果您熟悉Oracle：

- `pg_dump` ≈ `expdp`（匯出）
- `pg_restore` ≈ `impdp`（匯入二進位格式）
- `psql -f` ≈ `@script.sql`（執行SQL腳本）

## 總結

**關鍵差異是格式處理：**

- **pg_dump** 可以產生SQL文字檔或二進位檔
- **pg_restore** 只能處理二進位格式
- **純文字SQL檔必須用 psql 執行，不能用 pg_restore**

您現在理解為什麼剛才的 `roles.sql` 要用 `psql -f` 而不是 `pg_restore` 了嗎？