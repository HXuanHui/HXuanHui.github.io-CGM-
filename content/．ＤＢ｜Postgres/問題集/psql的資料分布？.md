#postgreSQL 
## PostgreSQL在Ubuntu中的資料存放位置

在Ubuntu系統中，PostgreSQL安裝完成後，資料會存放在以下預設位置：

**主要資料目錄（Data Directory）：**

- **預設路徑：** `/var/lib/postgresql/{版本號}/main/`
- 例如PostgreSQL 14版本：`/var/lib/postgresql/14/main/`
- 例如PostgreSQL 15版本：`/var/lib/postgresql/15/main/`

**具體的檔案分佈：**

1. **資料庫檔案：** `/var/lib/postgresql/{版本號}/main/base/`
    
    - 每個資料庫都有自己的子目錄，以OID命名
2. **WAL日誌檔案：** `/var/lib/postgresql/{版本號}/main/pg_wal/`
    
    - 儲存預寫式日誌（Write-Ahead Log）
3. **配置檔案：** `/etc/postgresql/{版本號}/main/`
    
    - `postgresql.conf` - 主要配置檔案
    - `pg_hba.conf` - 客戶端認證配置檔案
    - `pg_ident.conf` - 身分對應配置檔案
4. **日誌檔案：** `/var/log/postgresql/`
    
    - PostgreSQL的運行日誌

**如何確認資料目錄位置：**

您可以使用以下方法來確認實際的資料目錄：

```sql
-- 連線到PostgreSQL後執行
SHOW data_directory;
```

或者在系統命令列中：

```bash
sudo -u postgres psql -c "SHOW data_directory;"
```

**權限設定：**

- 資料目錄的擁有者是 `postgres` 使用者
- 權限通常設定為 `700`，確保只有postgres使用者可以存取

**補充說明：**

與您教材中提到的Windows環境不同，Linux系統遵循FHS（Filesystem Hierarchy Standard）標準，將不同類型的檔案分散存放：

- 資料檔案放在 `/var/lib/`
- 配置檔案放在 `/etc/`
- 日誌檔案放在 `/var/log/`

這種分散式的檔案配置提供了更好的系統管理和安全性，但對初學者來說可能需要適應這種與Windows不同的檔案組織方式。

如果您需要修改資料目錄位置，需要在初始化資料庫時使用 `initdb` 命令指定，或者修改配置檔案中的 `data_directory` 參數。


# 複習與問題
1. [psql檔案命名規則？](psql檔案命名規則？.md)