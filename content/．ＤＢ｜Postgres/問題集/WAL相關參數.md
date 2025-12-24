#postgreSQL 
## WAL 內容與級別

`wal_level` 決定 WAL 記錄的詳細程度，可選 `minimal`（基本恢復）、`replica`（支援複製）或 `logical`（邏輯解碼），預設 `replica`，需重啟生效 。
`wal_log_hints` 啟用提示位元記錄頁面內容變更，用於加速恢復 。
`full_page_writes` 在檢查點後首次修改頁面時記錄完整頁面，預設開啟以防硬體故障 。
## WAL 大小與保留

|參數|用途|預設值|備註|
|---|---|---|---|
|`max_wal_size`|兩個檢查點間 WAL 最大大小，控制檢查點頻率|1GB|提高減少檢查點但延長恢復 csdn+1​|
|`wal_keep_size`|至少保留 WAL 大小，用於備援或 PITR（取代 `wal_keep_segments`）|0|確保檔案不被刪除 pgsql+1​|
|`wal_buffers`|WAL 專用緩衝區大小|-1（shared_buffers 的 1/32）|影響寫入效能 [csdn](https://blog.csdn.net/qyj19920704/article/details/154508519)​|

## 檢查點控制

`checkpoint_timeout` 設定檢查點最大間隔時間，預設 5 分鐘 。
`checkpoint_completion_target` 控制檢查點完成目標百分比，預設 0.9 以分散 I/O 。
`checkpoint_segments`（舊版）類似 `max_wal_size`，決定 WAL 大小觸發檢查點 。​

## 寫入與同步

`wal_writer_delay` WAL 寫入器延遲，預設 200ms，用於調節頻率 。
`fsync` 確保 WAL 同步磁碟，預設開啟保證耐久性 。
`synchronous_commit` 控制提交時 WAL 同步方式（on/off/remote_apply 等） 。​

## 歸檔與複製

`archive_mode` 啟用 WAL 歸檔（off/on/always），預設 off，不產生 archive log 即使 `wal_level=replica` 。
`archive_command` 指定歸檔腳本，將 WAL 複製至外部儲存支援 PITR 。
WAL 歸檔獨立於 `wal_level`，未啟用時檔案僅在 pg_wal/ 循環重用 。
