---
theme:
  - default
class: 
paginate: true
header: PostgreSQL管理工具
---
<style scoped>h1 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size:60px
}
p { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size: 16px;
}
</style>

# PostgreSQL管理工具
黃暄惠 2024/11

---
## PostgreSQL管理工具
 - pg_
 - psql
 - pgAdmin
 - Postgres Enterprise Manager, PEM

**🚨PostgreSQL Instance的啟動與停止只能使用pg_ctl，無法藉由其他工具。**

---
<style>h2 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size:40px
}
</style>
## pg_
**舉例：**
1. pg_ctl：用於啟動、停止、重啟 PostgreSQL 伺服器。
2. pg_basebackup：用於對正在運行的 PostgreSQL 資料庫進行基礎備份。
3. [pg_dump](dump跟restore差在哪裡？.md)：用於備份單一資料庫。
4. [pg_restore](dump跟restore差在哪裡？.md)：從 `pg_dump` 創建的備份文件中恢復資料。

⭐ `pg_` 工具都是在命令行中使用的，直接在終端機中輸入命令來執行相應的操作。

---

## psql

⭐ 用於執行 SQL 查詢和管理資料庫。
### 基本功能

- **交互式查詢**: 使用者可以在 `psql` 中輸入 SQL 查詢，並即時查看結果。這使得資料庫的操作變得直觀且方便。
- **元命令支持**: 除了標準 SQL 命令外，`psql` 還支持以反斜線（`\`）開頭的元命令，這些命令用於管理資料庫和顯示幫助資訊。例如，`\h` 可以顯示 SQL 語法的幫助，而 `\q` 用於退出 `psql`。

---
### 連接到資料庫

要使用 `psql` 連接到資料庫，通常需要指定以下參數：

- `h`: 主機名稱（例如：localhost）
- `U`: 使用者名稱
- `W`: 提示輸入密碼
- `d`: 資料庫名稱

例如，連接到名為 `mydb` 的資料庫可以使用以下命令：

```bash
psql -h localhost -U username -d mydb
```

如果未指定資料庫名稱，則預設會使用當前使用者的名稱作為資料庫名稱。


---
### 常用元命令

以下是一些常用的元命令：

- **顯示幫助**: `\?`
- **顯示資料表**: `\dt`
- **查看當前使用者**: `SELECT current_user;`
- **退出 psql**: `\q`
- **顯示特定函數的定義:** `\sf` 
- **列出所有函數**: `\df`

---
### 系統資訊函數

1. **pg_reload_conf**：重新加載 PostgreSQL 配置。
2. **pg_encoding_to_char**：將編碼整數轉換為字符編碼名稱。
3. **pg_get_functiondef**：獲取函數的定義。
  
這將列出所有可用的函數，包括內建和用戶自定義的。


---
<style scoped>h2 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size:40px
}
</style>
## pgAdmin

常用功能
1. **用戶管理**：可以創建和管理數據庫用戶，避免使用 admin 帳號以提高安全性。
2. **數據庫對象管理**：支持創建、查看和編輯各種 PostgreSQL 對象，如表、索引、視圖等。
3. **查詢執行**：使用查詢工具執行 SQL 語句，並可導出結果。
4. **備份與恢復**：提供簡單的備份和恢復功能，可以選擇輸出格式

---

### 安裝與設置

pgAdmin 的最新版本為 pgAdmin 4
1. 從官方網站下載適合各平台的安裝包。
2. 通常只需點擊幾次“下一步”就可以完成。
3. 首次運行時需要設置密碼以保護訪問。
4. pgAdmin會在\pgsql\pgAdmin 4\runtime資料夾下

---

### 設置中文界面

在 pgAdmin 中，通過“File”菜單中的“Preferences”選項來設置語言為中文，然後保存設置並重新啟動即可生效。
![](．紀錄｜PostgreSQL/picture/{4FB38A7F-7CF2-4BEF-BE7D-506862BB8EC4}.png)![width:500px height:300px](．紀錄｜PostgreSQL/picture/{92ABA952-E3E1-4169-B9C1-01C9C0C2E4BF}.png)


---
<style scoped>h2 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size:40px
}
</style>
## Postgres Enterprise Manager
- **功能**:
    - **監控**: 能夠監控多個 PostgreSQL 實例及其性能。
    - **警報**: 提供即時警報功能，幫助用戶及時處理問題。
    - **調優**: 包含調優向導，建議最佳配置選項以提升性能。
---

- **組件**:
    - **PEM Server**: 包含 PostgreSQL 實例和 Apache 網頁伺服器，提供網頁介面。
    - **PEM Agent**: 負責收集和傳送監控數據，可安裝於多個伺服器上。
    - **PEM Web Client**: 瀏覽器介面，便於用戶管理和訪問 PEM 功能。
    ![width:500px height:400px](．紀錄｜PostgreSQL/picture/image4.png)
---

- **附加工具**:
    - **SQL Profiler**: 用於記錄 SQL 查詢計劃和性能數據的可選功能。
    - **查詢工具**: 提供交互式開發環境，允許用戶執行即時 SQL 查詢。
---

- **兼容性**: 支援多種操作系統和 PostgreSQL 版本，適合各種環境使用。
- **費用:**
    - 試用60天（2018資訊）
    - 代理商歐立威科技股份有限公司
---

### 結語
有趁手的工具之後可以連接本地的Server，遠端連線還須設定。

---

# 複習與問題
1. [dump跟restore差在哪裡？](dump跟restore差在哪裡？.md)

### Reference
1. [PostgreSQL：文檔：9.1：系統資訊函數](https://www.postgresql.org/docs/9.1/functions-info.html)
2. [透過 pgAdmin 操作 PostgreSQL 資料庫](https://learningsky.io/use-postgresql-databases-with-the-pgadmin/)
3. [pgAdmin介面操作](https://ithelp.ithome.com.tw/articles/10247750)
### 相關文件
1. PEM安裝

