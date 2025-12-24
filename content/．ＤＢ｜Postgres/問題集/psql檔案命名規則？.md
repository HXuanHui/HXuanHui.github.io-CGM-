#postgreSQL 

## PostgreSQL目錄數字的含義

### 1. **這些數字是OID (Object Identifier)**

PostgreSQL中的每個資料庫都有一個唯一的OID，這些數字就是資料庫的OID：

```sql
-- 查詢資料庫的OID
SELECT datname, oid FROM pg_database;

-- 常見的結果：
--     datname     |  oid  
-- ----------------+-------
--  template1      |     1
--  template0      |     0  
--  postgres       | 13067
--  my_database    | 16384
```

### 2. **OID的分配規則**

**系統預設的OID：**

- `1` = template1 (系統範本資料庫)
- `0` = template0 (唯讀範本資料庫)
- `postgres` = 通常是13067或類似數字

**使用者建立的資料庫：**

- 從16384開始依序分配
- 16384, 16385, 16386, 16387...

### 3. **為什麼從16384開始？**

PostgreSQL將OID空間分為幾個區間：

```
0-16383     : 系統保留OID (包含內建函數、系統表等)
16384以上   : 使用者物件OID (資料庫、表格、索引等)
```

這是PostgreSQL的設計原則，確保系統物件和使用者物件的OID不會衝突。

### 4. **同一個資料庫是否在同一目錄？**

**是的，除非指定tablespace，否則同一個資料庫的所有物件都在同一個目錄下：**

```bash
/var/lib/postgresql/14/main/base/
├── 1/              # template1
├── 13067/          # postgres資料庫
│   ├── 1247        # pg_type系統表
│   ├── 1249        # pg_attribute系統表
│   ├── 16385       # 使用者建立的表格
│   └── 16386       # 使用者建立的索引
└── 16384/          # 使用者資料庫 
    ├── 1247        # 系統表的本地副本
    ├── 16400       # 使用者表格1
    └── 16401       # 使用者表格2
```

### 5. **查詢檔案對應關係**

您可以用以下方式查詢：

```sql
-- 查詢表格對應的檔案
SELECT 
    schemaname,
    tablename,
    pg_relation_filepath(schemaname||'.'||tablename) as file_path
FROM pg_tables 
WHERE schemaname = 'public';

-- 查詢當前資料庫的OID
SELECT current_database(), oid 
FROM pg_database 
WHERE datname = current_database();
```

### 6. **Tablespace的例外情況**

只有當您明確指定tablespace時，檔案才會放在別的位置：

```sql
-- 建立自定義tablespace
CREATE TABLESPACE my_space LOCATION '/data/my_tablespace';

-- 建立表格到指定tablespace
CREATE TABLE my_table (id int) TABLESPACE my_space;
```

此時檔案會在：

```bash
/data/my_tablespace/PG_14_202107181/  # PostgreSQL版本目錄
└── 16384/                           # 資料庫OID目錄
    └── 16500                        # 表格檔案
```

### 7. **與Oracle的對比**

|特性|PostgreSQL|Oracle|
|---|---|---|
|**目錄命名**|使用OID數字|使用邏輯名稱|
|**檔案組織**|一個DB一個目錄|Tablespace概念|
|**預設位置**|都在base/OID/下|分散在不同tablespace|
|**系統管理**|依賴檔案系統|Oracle內部管理|

### 8. **實際查看範例**

```bash
# 在PostgreSQL資料目錄中
$ ls -la /var/lib/postgresql/14/main/base/
drwx------ postgres postgres     1
drwx------ postgres postgres 13067
drwx------ postgres postgres 16384

# 查看特定資料庫目錄
$ ls -la /var/lib/postgresql/14/main/base/16384/
-rw------- postgres postgres  1247    # pg_type
-rw------- postgres postgres  1249    # pg_attribute  
-rw------- postgres postgres 16385    # 使用者表格
-rw------- postgres postgres 16386    # 使用者索引
```

**總結：**

- 數字是資料庫的OID，自動分配
- 同一個資料庫所有檔案預設都在同一個OID目錄下
- 只有指定tablespace時才會例外
- 這種設計比Oracle簡單，但彈性較少