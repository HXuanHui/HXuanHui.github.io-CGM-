---
title: stanza info不連號
draft: false
maturity: stone
---

```
stanza: cg_test_product
    status: ok
    cipher: none

    db (current)
        wal archive min/max (16): 000000010000000000000003/000000040000000000000009

        full backup: 20251212-062816F
            timestamp start/stop: 2025-12-12 06:28:16+00 / 2025-12-12 06:28:20+00
            wal start/stop: 000000010000000000000007 / 000000010000000000000007
            database size: 36.4MB, database backup size: 36.4MB
            repo1: backup set size: 4.4MB, backup size: 4.4MB

        diff backup: 20251212-062816F_20251229-072007D
            timestamp start/stop: 2025-12-29 07:20:07+00 / 2025-12-29 07:20:11+00
            wal start/stop: 000000040000000000000009 / 000000040000000000000009
            database size: 36.4MB, database backup size: 4.9MB
            repo1: backup set size: 4.5MB, backup size: 610.8KB
            backup reference list: 20251212-062816F
            
        incr backup: 20251212-062816F_20251229-073546I
            timestamp start/stop: 2025-12-29 07:35:46+00 / 2025-12-29 07:35:48+00
            wal start/stop: 00000004000000000000000B / 00000004000000000000000B
            database size: 36.4MB, database backup size: 8.3KB
            repo1: backup set size: 4.5MB, backup size: 498B
            backup reference list: 20251212-062816F, 20251212-062816F_20251229-072007D
```


 **wal archive 的範圍：**

000000010000000000000003 / 000000040000000000000009

- **前 8 位數是「時間軸 (Timeline ID)」**：
    
    -  Full Backup 在 `Timeline 1` (`00000001...`)。
        
    - 最新的 Diff Backup 在 `Timeline 4` (`00000004...`)。
        
- **中間 8 位數與後 8 位數是「序號」**：
    
    - 從 Timeline 1 跳到 Timeline 4，中間顯然發生了 **3 次資料庫狀態的重大變更**。
	    
	- **Timeline 1**：你最初建立 Full Backup 的時候。
	    
	- **Timeline 2 & 3**：中間執行過還原測試，或是嘗試修復資料庫，每次還原成功並啟動後，Timeline 就會加 1。
	    
	- **Timeline 4**：你目前資料庫正在運行的狀態。
    

**我該擔心「不連號」嗎？**

**只要 pgBackRest 顯示 `status: ok`，就不需要擔心。**

- **自動處理分支**：pgBackRest 會自動處理這種 Timeline 切換。當你要求還原到 Timeline 4 的某個時間點時，它知道要先拉取 Timeline 1 的 Full Backup，然後依序重播 Timeline 1 的 WAL，直到分叉點，再轉向重播 Timeline 4 的 WAL。
    
- **安全性**：你的 Diff Backup (`...072007D`) 的 `wal start/stop` 是在 `Timeline 4`。這代表這個差異備份是完全基於當前這個「分叉」後的狀態，它是安全的。
	
- **自動Switch WAL**：當 pgBackRest 完成檔案拷貝後，它會呼叫 PostgreSQL 的 `pg_stop_backup()`（或類似內部函數）。**這個動作會強制執行一次 `pg_switch_wal()`**。



**實戰建議：如果您現在要還原**
因為您有 Timeline 切換，如果要進行 PITR，建議在指令中多加一個參數，避免[這種](https://www.modb.pro/db/1906926193694224384)時間線的以外增加而導致無法復原期望的時間點。


```
# 加入 --target-timeline=current 確保它沿著最新的路徑走
sudo -u postgres pgbackrest --stanza=cg_test_prodcut --type=time \
  --target="2025-12-29 07:20:00" --target-timeline=current restore
```


### Reference
1. [官方指令與參數說明](https://pgbackrest.org/command.html)
2. [target-timeline使用案例](https://www.modb.pro/db/1906926193694224384)


#postgreSQL 
