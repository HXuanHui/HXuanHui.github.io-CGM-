#postgreSQL 
## PostgreSQL Block Size 調整的正確方法

### 1. **調整方式**

PostgreSQL的block size需要在**編譯時期**指定，使用configure參數：

```bash
./configure --with-blocksize=32768 --with-wal-blocksize=32768
make
make install
```

### 2. **可用的Block Size選項**

- 1KB, 2KB, 4KB, 8KB (預設), 16KB, 32KB
- 必須是2的倍數
- 預設值是8192 bytes (8KB)

### 3. **查看目前Block Size**

```sql
-- 查看目前的block size
SHOW block_size;

-- 或者查看相關參數
SELECT name, setting FROM pg_settings 
WHERE name IN ('block_size', 'wal_block_size');
```

### 4. **實際性能測試結果**

根據搜尋到的深度測試：在SSD環境下，使用32KB block size比8KB在分析型查詢上有顯著性能提升

**測試結果顯示**：

- 8KB: 執行時間變化不大
- 16KB: 第4次執行開始時間大幅縮短
- 32KB: 第2次執行就開始時間大幅縮短

### 5. **為什麼要調整Block Size？**

**大Block Size優勢（16KB/32KB）**：

- **分析型查詢**：大量循序讀取時更有效率
- **SSD環境**：配合SSD的特性，減少I/O次數
- **數據倉儲**：大表掃描性能更好

**小Block Size優勢（8KB或更小）**：

- **OLTP工作負載**：隨機讀寫較多的交易型應用
- **記憶體使用**：減少不必要的資料載入
- **並發性**：減少鎖競爭

### 6. **與Oracle的比較**

| 項目    | PostgreSQL         | Oracle          |
| ----- | ------------------ | --------------- |
| 調整時機  | 編譯時設定              | 建立資料庫時設定        |
| 調整方法  | `--with-blocksize` | `db_block_size` |
| 動態調整  | 不可（需重編譯）           | 不可（建立後固定）       |
| 表空間層級 | 無法個別設定             | 可用非標準block size |

### 7. **實務建議**

**何時考慮調整**：

```sql
-- 如果是分析型工作負載
-- 考慮使用 16KB 或 32KB

-- 如果是OLTP工作負載  
-- 保持預設 8KB
```

**注意事項**：

- 需要重新編譯PostgreSQL
- 無法線上調整
- 影響整個instance的所有資料庫
- 可能遇到未知的bug（很少人使用非標準大小）

