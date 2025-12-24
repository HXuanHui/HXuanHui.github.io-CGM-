---
theme: gaia
class:
paginate: true
header: 備份還原操作
style: |-
  section {
    font-size: 28px;
    padding: 90px;
  }
  h1 {
    display: flex; 
    justify-content: center;
    align-items: center;
    color: #008ED1; /* Oracle red */
    font-size:80px;
  }
  h2 {
    color: #008ED1; /* Oracle red */
  }
  ul, ol {
    list-style-position: inside;
  }
  img{display: block; margin:0 auto;object-fit: contain;
  }
  table{font-size:24px}
---

# 環境準備與前置檢查

在安裝任何軟體之前，進行系統性的環境驗證是預防常見部署失敗、確保後續流程順暢無誤的關鍵步驟。這些「起飛前檢查」將協助您確認所有系統與 PostgreSQL 資料庫的先決條件均已滿足。


----

### 系統環境確認 

``` bash
# 查看發行版本詳細資訊
cat /etc/os-release

# 查看 CPU 架構 (x86_64 還是 aarch64)
uname -m
```
---

### PostgreSQL資料路徑 

執行以下指令來查找 PostgreSQL 主程序的詳細資訊，並找出其資料目錄 (Data Directory) 的真實路徑
- **指令：**
``` bash
# 如果資料庫正在執行中，這是最準確的方法
ps -ef | grep postgres | grep -v grep
```
- **範例輸出：**`/usr/pgsql-14/bin/postgres -D /var/lib/pgsql/14/data`

- **關鍵看點：** 請在輸出中尋找 `-D` 參數後面的路徑。在此範例中，真實資料路徑為 `/var/lib/pgsql/14/data`。此路徑在後續設定 `pgbackrest.conf` 時至關重要。

---
### 操作畫面 

![width:600px ](．紀錄｜PostgreSQL/picture/postgresEnv.png)
* **OS:** Ubuntu 24.04 LTS (Noble Numbat)
* **DB:** PostgreSQL 16.9
* **相關路徑 ：**
  * 設定檔路徑：`/etc/postgresql/16/main/postgresql.conf`
  * 真實資料路徑：`/var/lib/postgresql/16/main`
  * 服務名稱：`postgresql` (或 `postgresql@16-main`)

-----

## pgBackRest 安裝與基礎設定


安裝 `pgbackrest` 並建立一個用來存放備份檔的「倉庫 (Repository)」。

### 安裝 pgBackRest 套件
PostgreSQL 的官方 PGDG APT 儲存庫已包含 pgBackRest 套件，安裝過程非常直接。

```bash
sudo apt update
sudo apt install pgbackrest -y
```


----
### 建立與設定備份儲存庫 (Repository)

備份儲存庫是 pgBackRest 存放所有備份檔案與 WAL 歸檔的目錄。我們需要手動建立它，並設定正確的權限。


**建立備份儲存庫**
`/var/lib/pgbackrest` 資料夾會自動被建立，並將權限交給 `postgres` 使用者（因為資料庫是的執行身分）。

```bash
sudo mkdir -p /var/lib/pgbackrest 
sudo chown postgres:postgres /var/lib/pgbackrest
sudo chmod 750 /var/lib/pgbackrest
```

---
### 操作畫面

![width:1000px ](．紀錄｜PostgreSQL/picture/pgBackRestVer.png)


-----

### 設定 pgbackrest.conf

此設定檔是 pgBackRest 的大腦，它定義了備份儲存庫的位置、資料庫叢集的位置以及其他關鍵參數。

**1. 編輯設定檔**
開啟設定檔。

```bash
sudo vim /etc/pgbackrest/pgbackrest.conf
```
---

**2. 修改pgbackrest.conf**
根據 `ps -ef | grep postgres | grep -v grep` 結果，將路徑填入 `pg1-path` 。
這裡我們將 Stanza (備份設定檔名) 取名為 `cgh_main` 。以下修改為最低需求，詳細配置參考[pgBackRest參數設定](pgBackRest參數設定.md)。

```ini
[global]
# 指定備份倉庫路徑
repo1-path=/var/lib/pgbackrest
# 設定保留策略：保留最近 2 份全備份 (Lab 測試用，正式環境通常設 7-14)
repo1-retention-full=2
# 設定日誌級別
log-level-console=info
log-level-file=detail
# 啟用並行處理 (加速備份/還原)
process-max=2

[cgh_main]
# 資料庫真實路徑 (來自 ps -ef 結果)
pg1-path=/var/lib/postgresql/16/main
```

---


**3. 修正設定檔權限**
為了安全，這個檔案應該只有 postgres 能讀寫。

```bash
sudo chown postgres:postgres /etc/pgbackrest/pgbackrest.conf
sudo chmod 640 /etc/pgbackrest/pgbackrest.conf
```

-----

### 設定 PostgreSQL 開啟歸檔模式

這一步是讓 PostgreSQL 開始將其產生的 WAL 日誌傳送給 pgBackRest 進行歸檔管理，這是實現 PITR 的必要條件。

**1. 編輯 PostgreSQL 設定檔**

```bash
sudo vim /etc/postgresql/16/main/postgresql.conf
```
---

<style> blockquote { font-size: 0.6em; } </style>

**2. 修改以下參數**
請利用搜尋功能  找到這些參數並修改。如果前面有 `#` 註解符號，請拿掉。詳細說明請參考[WAL相關參數](WAL相關參數.md)。

```ini
# --- 修改內容 ---

# 1. 確保 WAL 級別是 replica (通常 16 版預設就是 replica，確認一下即可)
wal_level = replica
# 2. 開啟歸檔模式
archive_mode = on
# 3. 設定歸檔指令 (注意 stanza 名稱要跟上面一致：cgh_main)
archive_command = 'pgbackrest --stanza=cgh_main archive-push %p'
```
> 雖然上述設定能讓系統運作，但必須考慮極端情況：如果備份儲存庫發生問題，`archive_command` 可能會持續失敗，導致 `pg_wal` 目錄下的 WAL 檔案堆積，最終塞滿磁碟並導致資料庫崩潰。
> 為此，pgBackRest 提供了一個關鍵的安全閥：`archive-push-queue-max` 參數。此參數決定在危機中，優先考慮保持資料庫在線（透過丟棄 WAL 並中斷備份鏈），還是確保備份完整性（冒著磁碟滿載導致資料庫崩潰的風險）？對於大多數生產系統，優先考慮可用性是正確的營運決策。

---

**3. 重啟 PostgreSQL 使設定生效**

```bash
sudo systemctl restart postgresql
```

-----

### 初始化 Stanza 並驗證組態

現在兩邊的設定都已完成，我們需要執行最後的「握手」程序來初始化備份系統並驗證其連通性。

**1. 切換身分到 postgres**
接下來的操作都建議用 postgres 身分進行，避免權限問題。

```bash
sudo -u postgres -i
```

**2. 建立 Stanza (初始化)**
這會建立必要的目錄結構。

```bash
pgbackrest --stanza=cgh_main stanza-create
```

---
**3. 檢查設定 (Check)**
這是一個自我檢測指令，它會測試「能否寫入備份區」以及「能否讀取資料庫」。

```bash
pgbackrest --stanza=cgh_main check
```

當您在輸出結尾看到 `command end: completed successfully` 時，才代表您的基礎設定已成功完成。至此，一個基礎的備份系統已部署完成並通過驗證。


---
# 相關步驟與知識
1. [pgBackRest基礎還原操作](pgBackRest基礎還原操作.md)
2. [pgBackRest參數設定](pgBackRest參數設定.md)
3. [備份檔會放在哪裡？](備份檔會放在哪裡？.md)
