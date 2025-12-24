#postgreSQL 
## PostgreSQL的FSM和VM檔案

### 1. **FSM檔案 (Free Space Map)**

**FSM = Free Space Map (空閒空間映射)**

**作用：**

- 追蹤每個data page中有多少可用空間
- 幫助PostgreSQL快速找到有足夠空間插入新資料的page
- 避免掃描整個表格來尋找可用空間

**檔案命名：**

```bash
# 主要表格檔案
16384           # 表格的主要資料檔案

# FSM檔案 (在同一目錄下)
16384_fsm       # 對應的Free Space Map檔案
```

**內部結構：** FSM使用3個bit來記錄每個page的空閒空間等級：

- 0: 空閒空間 < 12.5%
- 1: 空閒空間 12.5% - 25%
- 2: 空閒空間 25% - 50%
- 3: 空閒空間 50% - 75%
- 4: 空閒空間 75% - 87.5%
- 5: 空閒空間 87.5% - 100%
- 6: 完全空的page
- 7: 保留值

### 2. **VM檔案 (Visibility Map)**

**VM = Visibility Map (可見性映射)**

**作用：**

- 追蹤哪些page只包含已提交的、對所有交易都可見的資料
- 優化VACUUM操作，跳過不需要檢查的page
- 加速Index-Only Scan

**檔案命名：**

```bash
16384_vm        # 對應的Visibility Map檔案
```

**內部結構：** VM使用2個bit來追蹤每個page的狀態：

- Bit 1: 該page是否只包含已凍結的tuples
- Bit 2: 該page是否只包含對所有交易都可見的tuples

### 3. **實際檔案結構範例**

```bash
# PostgreSQL資料目錄中的實際例子
/var/lib/postgresql/14/main/base/16384/
├── 16385           # 表格的主要資料
├── 16385_fsm       # 該表格的Free Space Map
├── 16385_vm        # 該表格的Visibility Map
├── 16386           # 索引的主要資料
├── 16386_fsm       # 該索引的Free Space Map (索引也有FSM)
├── 16387           # 另一個表格
├── 16387_fsm       # 另一個表格的FSM
└── 16387_vm        # 另一個表格的VM
```

### 4. **與Oracle的對比**

| 功能           | PostgreSQL | Oracle             |
| ------------ | ---------- | ------------------ |
| **空間管理**     | FSM檔案      | Bitmap Block (BMB) |
| **位置**       | 獨立檔案       | 存在segment內部        |
| **可見性追蹤**    | VM檔案       | 透過MVCC+SCN         |
| **VACUUM優化** | VM加速       | Undo segment管理     |

### 5. **FSM和VM的查詢方式**

```sql
-- 查詢表格的FSM資訊
SELECT * FROM pg_freespace('table_name');

-- 查詢表格的VM資訊  
SELECT * FROM pg_visibility_map('table_name');

-- 查詢表格檔案大小統計
SELECT 
    pg_size_pretty(pg_relation_size('table_name')) as table_size,
    pg_size_pretty(pg_relation_size('table_name', 'fsm')) as fsm_size,
    pg_size_pretty(pg_relation_size('table_name', 'vm')) as vm_size;
```

### 6. **重要的維護操作**

**VACUUM對FSM和VM的影響：**

```sql
-- 更新FSM，回收死亡tuples佔用的空間
VACUUM table_name;

-- 更新VM，標記clean pages
VACUUM table_name;

-- 強制更新FSM
VACUUM (FULL) table_name;
```

**重建FSM：**

```sql
-- 如果FSM損壞，可以重建
VACUUM table_name;
-- 或者
REINDEX TABLE table_name;
```

### 7. **效能影響**

**FSM的效能影響：**

- **好處：** 加速INSERT操作，快速找到有空間的page
- **壞處：** 需要額外的磁碟空間和維護開銷

**VM的效能影響：**

- **好處：** 大幅加速VACUUM和Index-Only Scan
- **壞處：** 每次page修改都需要更新VM

### 8. **故障排除**

**常見問題：**

```sql
-- 檢查表格膨脹情況
SELECT 
    schemaname,
    tablename,
    n_dead_tup,
    n_live_tup,
    round(n_dead_tup::float/n_live_tup::float*100,2) as dead_ratio
FROM pg_stat_user_tables
WHERE n_live_tup > 0;

-- 查看最後VACUUM時間
SELECT 
    relname,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables;
```

## 總結

**FSM (*_fsm檔案)：**

- 追蹤空閒空間，加速INSERT
- 類似Oracle的space management bitmap

**VM (*_vm檔案)：**

- 追蹤頁面可見性，優化VACUUM
- PostgreSQL獨有的MVCC優化機制

這些檔案是PostgreSQL實現高效能空間管理和MVCC的關鍵組件，雖然增加了檔案數量，但大幅提升了資料庫的運行效率。