#ORACLE #postgreSQL 

## Oracle 19c 的架構

從教材可以看到，Oracle 的連線架構是這樣的：

**Oracle 不是 instance 直接綁定 port**，而是透過 **Listener** 來處理連線：

1. **Listener** 綁定特定的 port（如預設的 1521）
2. **Instance** 透過 LREG process 向 Listener 註冊自己的服務
3. **PDB（可插拔資料庫）** 作為服務註冊到 Listener

從你的教材可以看到：

- `tnsping edvmr1p0:1521/orclpdb1` - 這裡的 1521 是 Listener 的 port
- 一個 Listener 可以服務多個 instance 和多個 PDB
- TNS Names 設定的是 `HOST:PORT/SERVICE_NAME` 的對應關係

## PostgreSQL 的架構

從教材的 PostgreSQL 部分可以看到，**PostgreSQL 確實是 instance 直接綁定 port**：

PostgreSQL 的 **instance（在 PG 中稱為 cluster）在 initdb 時就需要指定 port**：

### PostgreSQL port 設定時機：

1. **initdb 階段**：
    
    ```bash
    initdb -D /path/to/data --locale=C --encoding=UTF8
    ```
    
    此時會產生 `postgresql.conf`，裡面有 `port = 5432`
    
2. **啟動階段**：
    
    ```bash
    pg_ctl -D /path/to/data -l logfile start
    # 或
    postgres -D /path/to/data -p 5433  # 指定不同port
    ```
    
3. **設定檔修改**： 編輯 `postgresql.conf` 中的 `port` 參數
    

### 解決多個 instance 都用相同 port 的問題：

你說得對，如果用相同的設定多次執行 initdb，預設都會是 port 5432。解決方法：

1. **初始化時指定不同的 data directory**：
    
    ```bash
    initdb -D /data/pg1  # 第一個instance
    initdb -D /data/pg2  # 第二個instance
    ```
    
2. **修改各自的 postgresql.conf**：
    
    ```bash
    # /data/pg1/postgresql.conf
    port = 5432
    
    # /data/pg2/postgresql.conf  
    port = 5433
    ```
    
3. **啟動時指定不同 port**：
    
    ```bash
    pg_ctl -D /data/pg1 -o "-p 5432" start
    pg_ctl -D /data/pg2 -o "-p 5433" start
    ```
    

## 主要差異總結

|項目|Oracle|PostgreSQL|
|---|---|---|
|Port 綁定|Listener 綁定 port|Instance 直接綁定 port|
|連線方式|Client → Listener → Instance|Client → Instance|
|多資料庫支援|透過 PDB/Service name|透過不同 instance/port|
|Port 設定時機|Listener 設定檔|initdb 或 postgresql.conf|
