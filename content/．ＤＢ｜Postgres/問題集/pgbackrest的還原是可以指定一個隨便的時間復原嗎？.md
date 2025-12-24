#postgreSQL 
### 情境 1：要還原的時間點「還在本地 `pg_wal` 中」

這通常發生在：您剛誤刪資料，且該交易非常新，`pg_wal` 段還沒被切換（switch）或尚未被 pgBackRest 成功歸檔到遠端。

- **還原邏輯：** 結合 **pgBackRest 倉庫的數據** + **本地存活的 WAL 檔案**。
    
- **操作關鍵：**
    
    1. **保護現場：** 立刻停止資料庫，將 `/var/lib/pgsql/data/pg_wal/` 目錄整個複製出來（例如存到 `/tmp/wal_rescue`）。
        
    2. **執行 Restore：** 使用 pgBackRest 還原最近的一個備份。
        
    3. **手動補齊：** 還原後，將 `/tmp/wal_rescue` 裡面的所有檔案拷貝回還原後的 `pg_wal/` 目錄中。
        
    4. **設定目標：** 在還原參數中加入 `--type=time` 指定精確的誤刪前時間點。
        
- **結果：** 資料庫會先從 Repository 下載 Archive WAL，最後銜接本地的 WAL 段，達成「零資料遺失」還原。
    

---

### 情境 2：只有一個 Full Backup（且 WAL 已歸檔）

這是最標準的 **PITR (Point-In-Time Recovery)** 情境。你能還原到多晚的時間，取決於你的 `archive-push` 是否成功將最後的 WAL 傳送到 repo。

- **還原邏輯：** 依靠 **Full Backup** 作為地基，再由 pgBackRest **自動拉取 Archive WAL** 進行重播。
    
- **操作關鍵：** 
    ```
    pgbackrest --stanza=your_db restore --type=time \
    --target="2025-12-23 11:00:00" --delta
    ```
    
- **結果：** 資料庫會恢復到該 Full Backup 狀態，然後一路執行 WAL 直到 11:00:00 停止。
    

---

### 情境 3：只有 Full Backup，但「沒有任何archived WAL」

如果您關閉了 `archive_mode`，或者 WAL 倉庫損毀了。

- **還原邏輯：** 只能還原到「備份完成的那一刻」。
    
- **限制：** 您無法指定任何備份時間點之後的時間。資料庫啟動後，數據會停留在備份結束時的一致性狀態（Consistent Point）。
    
- **缺點：** 備份之後到災難發生前的所有交易全部遺失。
    

---

### 情境 4：有多個備份（Full + Incremental/Differential）

如果您平時有做增量備份，pgBackRest 會非常聰明地處理。

- **還原邏輯：**
    
    1. 自動選擇距離目標時間點「最近且更早」的一個備份（可能是週日的 Full 或週二的 Incremental）。
        
    2. 只還原該備份後的差異檔案。
        
    3. 再重播剩下的 WAL。
        
- **優點：** 速度最快。比起只用 Full Backup，它減少了需要「重播」的 WAL 數量（重播 WAL 是單執行緒，通常比拷貝檔案慢）。
    

---

### 補充情境 5：Database Instance 完全毀損，且無本地 `pg_wal`

當整台伺服器起火或硬碟壞掉時。

- **還原邏輯：** 100% 依賴 pgBackRest 倉庫（Repository）。
    
- **操作關鍵：** 在新伺服器安裝同版本的 PostgreSQL，設定好 `pgbackrest.conf` 指向遠端倉庫（如 S3 或備份伺服器），直接執行 restore。
    
- **限制：** 您最多只能還原到「最後一個成功歸檔到倉庫」的 WAL 時間點。
    

---

### 總結對照表

| **情境**        | **必備組件**                                  | **還原精確度** | **資料遺失風險**    |
| ------------- | ----------------------------------------- | --------- | ------------- |
| **本地 WAL 還在** | Full Backup + Archive WAL + **本地 pg_wal** | 極高 (秒級)   | 近乎零           |
| **標準 PITR**   | Full Backup + **Archive WAL**             | 高 (秒級)    | 遺失最後未歸檔的部分    |
| **僅有備份檔**     | Full Backup                               | 僅限備份當下    | 遺失備份後的所有資料    |
| **增量備份**      | **Incremental** + Archive WAL             | 高 (秒級)    | 速度最快，風險同 PITR |

最後一個小提醒：

在執行還原前，強烈建議加上 --dry-run 參數，讓 pgBackRest 告訴您它打算怎麼還原、會用到哪些備份檔，確認無誤後再正式執行。

