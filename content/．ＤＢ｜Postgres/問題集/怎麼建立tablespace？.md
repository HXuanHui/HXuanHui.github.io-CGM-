#postgreSQL 
**必須手動建立目錄：**

1. **安全考量** - PostgreSQL不會自動建立目錄，避免誤建立錯誤路徑
2. **權限控制** - 需要確保postgres用戶有適當的讀寫權限
3. **檔案系統規劃** - 讓DBA可以事先規劃儲存結構

## 正確的建立步驟

```bash
# 1. 建立目錄（需要root或適當權限）
sudo mkdir -p /cgmpgst/table
sudo mkdir -p /cgmpgst/db

# 2. 設定擁有者和權限
sudo chown postgres:postgres /cgmpgst/table /cgmpgst/db
sudo chmod 700 /cgmpgst/table /cgmpgst/db
```

```sql
-- 3. 建立tablespace
CREATE TABLESPACE cgmpgst_table
  OWNER postgres
  LOCATION '/cgmpgst/table';

CREATE TABLESPACE cgmpgst_db
  OWNER postgres
  LOCATION '/cgmpgst/db';
```

## 常見錯誤情況

如果目錄不存在或權限不正確，會出現這些錯誤：

- `could not set permissions on directory "/path": No such file or directory`
- `directory "/path" does not exist`
- `could not set permissions on directory "/path": Permission denied`

## 與Oracle的差異

這點確實與Oracle不同：

- **Oracle**: 可以透過OMF(Oracle Managed Files)自動建立目錄和檔案
- **PostgreSQL**: 必須手動建立目錄，PostgreSQL只負責在指定目錄下建立資料檔案

## 最佳實務建議

1. **事先規劃目錄結構**

```bash
/data/postgres/
├── tablespaces/
│   ├── app_data/
│   ├── app_index/
│   └── temp_data/
```

2. **統一權限設定**

```bash
sudo chown -R postgres:postgres /data/postgres/
sudo chmod -R 700 /data/postgres/
```

3. **檢查目錄是否為空**
    - PostgreSQL要求目錄必須存在且為空
    - 如果目錄內有檔案會建立失敗

這個設計讓PostgreSQL的tablespace管理更加明確和安全，雖然需要額外步驟，但能避免很多潛在的檔案系統問題。