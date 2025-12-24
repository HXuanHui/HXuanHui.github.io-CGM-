---
theme:
  - default
class:
paginate: true
header: 備份還原操作
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
}%%  %%
</style>


## 執行備份與檢視備份資訊
在成功設定並驗證系統後，將執行不同類型的備份，並學習如何使用 `info` 指令來檢視與理解備份狀態。

### 執行首次完整備份 (Full Backup)

第一次備份必須是完整備份，它建立了後續所有差異或增量備份的基礎。

```bash
# (目前身分仍是 postgres)
pgbackrest --stanza=cgh_main --type=full backup
```

**提示：** 如果您在執行備份時不指定 `--type` 參數，pgBackRest 預設會嘗試執行增量備份 (`incr`)。然而，在沒有任何現存備份的情況下，它會自動將類型轉換為完整備份 (`full`)。


---

### 執行差異與增量備份

在有了完整備份之後，就可以執行更節省空間與時間的備份類型。

- **執行差異備份 (Differential Backup)** 此備份包含自上次完整備份以來的所有變更。
 ```bash
pgbackrest --stanza=cgh_main --type=diff backup
```
- **執行增量備份 (Incremental Backup)** 此備份僅包含自上一次備份（不論類型）以來的所有變更。
 ```bash
pgbackrest --stanza=cgh_main --type=incr backup
```

---

### 檢視備份資訊

`info` 指令是檢視備份儲存庫狀態的統一入口，它會清晰地列出所有可用的備份集。

```bash
sudo -u postgres pgbackrest info
```

----

將會看到類似以下的範例輸出：

```
stanza: cgh_main
    status: ok
    cipher: none

    db (current)
        wal archive min/max (17): 000000010000000000000001/000000010000000000000005

    full backup: 20251018-084114F
        timestamp start/stop: 2025-10-18 08:41:14+00 / 2025-10-18 08:41:18+00
        wal start/stop: 000000010000000000000002 / 000000010000000000000003
        database size: 22MB, database backup size: 22MB
        repo1: backup set size: 2.9MB, backup size: 2.9MB

    diff backup: 20251018-084114F_20251018-084118D
        timestamp start/stop: 2025-10-18 08:41:18+00 / 2025-10-18 08:41:20+00
        wal start/stop: 000000010000000000000004 / 000000010000000000000005
        database size: 22MB, database backup size: 8.3KB
        repo1: backup set size: 2.9MB, backup size: 448B
        backup reference total: 1 full
```



---

```
stanza: cgh_main
    status: ok
    cipher: none

    db (current)
        wal archive min/max (17): 000000010000000000000001/000000010000000000000005

```

以下是輸出中關鍵資訊的解析：

- `status: ok`: 表示此 Stanza 的狀態正常，可以進行備份與還原。
- `wal archive min/max`: 顯示目前儲存庫中已歸檔的 WAL 檔案的最小與最大序號範圍。
---

以下是輸出中關鍵資訊的解析：

```
    full backup: 20251018-084114F
        timestamp start/stop: 2025-10-18 08:41:14+00 / 2025-10-18 08:41:18+00
        wal start/stop: 000000010000000000000002 / 000000010000000000000003
        database size: 22MB, database backup size: 22MB
        repo1: backup set size: 2.9MB, backup size: 2.9MB

```

- `full backup` **/** `diff backup`: 列出所有可用的備份集。每個備份集都有一個唯一的標籤（Label），並包含其執行時間戳、所需的 WAL 範圍等詳細資訊。
- `database size` **vs.** `database backup size`: `database size` 是備份當下資料庫的實際總大小，而 `database backup size` 是該次備份實際需要傳輸的資料量。對於差異或增量備份，後者會遠小於前者。


---


以下是輸出中關鍵資訊的解析：
```
    diff backup: 20251018-084114F_20251018-084118D
        timestamp start/stop: 2025-10-18 08:41:18+00 / 2025-10-18 08:41:20+00
        wal start/stop: 000000010000000000000004 / 000000010000000000000005
        database size: 22MB, database backup size: 8.3KB
        repo1: backup set size: 2.9MB, backup size: 448B
        backup reference total: 1 full
```


- `repo backup set size` **vs.** `backup size`: `repo backup set size` 指的是要還原此備份所需的總資料量（包含其依賴的所有備份），而 `backup size` 僅指該次備份本身在儲存庫中的大小（已壓縮）。

既然我們已經擁有了有效的備份集，下一步便是透過模擬災難來驗證其可用性，確保在真正需要時能夠成功還原。


---

## 災難復原演練：時間點復原 (PITR)
### 連線資料庫並建立測試資料

我們寫入一筆資料，標記為「重要資料」。

```bash
psql -c "CREATE TABLE patient_data (id serial, name text, status text,
 created_at timestamp);"
psql -c "INSERT INTO patient_data (name, status, created_at) VALUES ('王小明', 
'ICU', now());"

# 確認資料存在，並記下現在時間
psql -c "SELECT * FROM patient_data;"
date
```

> **記下這裡顯示的時間**。


![](Pasted%20image%2020251212163827.png)


---
###  模擬災難 


```bash
psql -c "DROP TABLE patient_data;"
psql -c "SELECT * FROM patient_data;" 
# 此時應該會報錯：ERROR: relation "patient_data" does not exist
```

其他的情境：[pgbackrest復原情境](pgbackrest的還原是可以指定一個隨便的時間復原嗎？.md)


---
### 執行時間點還原 (PITR)

**還原必須先停止資料庫服務。**

#### **退出 postgres 使用者 (回到 cghadmin)**

```bash
exit
```

#### **停止資料庫**

```bash
sudo systemctl stop postgresql
```
---

#### **執行還原 (指定回到災難發生前的時間)**
還原到剛剛紀錄的時間再晚一點時候。更多還原參數說明，參考[pgBackRest參數設定](pgBackRest參數設定.md)

```bash
# 請將下方的時間替換為您剛剛記下的時間再往後一點
sudo -u postgres pgbackrest --stanza=cgh_main --delta --type=time --target="2025-12-12 08:30:00" restore
```
- **參數解析**:
        - `--delta`: 這是最佳實踐，它能智慧地同步資料目錄，只還原變動或遺失的檔案，比完全清空後再還原更快也更安全。
        - `--type=time`: 指定執行時間點復原。
        - `--target`: 指定還原到的目標時間。

---

#### **啟動資料庫**

```bash
sudo systemctl start postgresql
sudo -u postgres psql -c "SELECT pg_wal_replay_resume();"
```
#### **驗證復原結果**
檢查資料是否回來了。

```bash
sudo -u postgres psql -c "SELECT * FROM patient_data;"
```

---
## 自動化排程：設定 Cron

手動執行備份容易出錯且不可靠。我們應使用 cron 來設定自動化的備份排程，確保備份定期、一致地執行。

1. **編輯 postgres 使用者的 crontab**
2. **加入備份排程範例** 以下範例設定了每週日凌晨 2 點執行一次完整備份，週一至週六每天凌晨 2 點執行一次增量備份。將 stdout (`>>`) 和 stderr (`2>&1`) 的輸出重新導向至日誌檔是生產自動化的強制要求。它為成功的備份提供了不可或缺的稽核軌跡，並且是診斷自動化排程失敗的主要來源。


---
## 監控與日誌檢視
有效的監控與日誌分析是問題排查與確保系統健康的基礎。

- **pgBackRest 日誌**: pgBackRest 的主要日誌檔案預設存放於 `/var/log/pgbackrest/`。每個 Stanza 和命令都會有獨立的日誌檔，例如 `cgh_main-backup.log`。當備份或還原操作失敗時，這裡是最先需要檢查的地方。為了進行有效的故障排除，強烈建議在 `pgbackrest.conf` 中設定 `log-level-file=detail`，這能在不影響主控台輸出的情況下，提供用於事後分析的詳細日誌。
- **PostgreSQL 日誌**: 有時問題的根源在於資料庫本身，例如歸檔失敗。您應同時檢查 PostgreSQL 的系統日誌來獲取更全面的資訊。

在遇到問題時，結合檢查 pgBackRest 與 PostgreSQL 的日誌，通常能快速定位問題所在。