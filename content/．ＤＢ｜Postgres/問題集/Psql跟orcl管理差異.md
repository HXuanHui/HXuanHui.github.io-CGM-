#ORACLE #postgreSQL 
## PostgreSQL依賴檔案系統的管理方式

### 1. **Tablespace管理差異**

**Oracle的做法：**

```sql
-- Oracle需要明確指定datafile的大小、位置、擴展方式
CREATE TABLESPACE demots 
DATAFILE '/u01/app/oracle/oradata/ORCLCDB/demots_01.dbf' 
SIZE 10M 
AUTOEXTEND ON 
NEXT 1M 
MAXSIZE 1G
EXTENT MANAGEMENT LOCAL 
UNIFORM SIZE 64K;
```

**PostgreSQL的做法：**

```sql
-- PostgreSQL只需要指定目錄路徑，其他交給檔案系統
CREATE TABLESPACE my_tablespace 
LOCATION '/var/lib/postgresql/tablespaces/my_data';
```

### 2. **檔案大小管理**

**Oracle：**

- 內建精細的空間管理：SMALLFILE/BIGFILE tablespace
- 從您教材看到：`一個表格空間最多可由2^10個datafile組成，每個datafile最多可以有2^22個data block`
- Oracle自己管理extent (64K, 1M, 8M, 64M)
- 需要設定AUTOEXTEND, NEXT, MAXSIZE等參數

**PostgreSQL：**

- 沒有這些複雜的限制，直接依賴檔案系統
- 檔案會根據需要自動增長，受限於檔案系統限制
- 不需要預先規劃檔案大小

### 3. **實際例子對比**

**Oracle建立tablespace的複雜度：**

```sql
-- 需要考慮很多Oracle特有的參數
CREATE TABLESPACE sales_data
DATAFILE '/u01/app/oracle/oradata/ORCLCDB/sales01.dbf' SIZE 100M,
         '/u01/app/oracle/oradata/ORCLCDB/sales02.dbf' SIZE 100M
AUTOEXTEND ON NEXT 10M MAXSIZE 500M
EXTENT MANAGEMENT LOCAL UNIFORM SIZE 1M
SEGMENT SPACE MANAGEMENT AUTO;
```

**PostgreSQL建立tablespace的簡潔度：**

```sql
-- 只需要指定路徑，其他都靠檔案系統
CREATE TABLESPACE sales_data LOCATION '/data/sales';
```

### 4. **檔案系統層級管理的具體表現**

**PostgreSQL的檔案結構：**

```bash
# PostgreSQL直接使用檔案系統目錄結構
/var/lib/postgresql/14/main/
├── base/               # 各資料庫目錄
│   ├── 1/             # template1
│   ├── 13067/         # postgres
│   └── 16384/         # 使用者資料庫
├── pg_wal/            # WAL檔案
├── pg_xact/           # 交易狀態檔案
└── tablespaces/       # 自定義tablespace的符號連結
```
```sql
-- 查詢資料庫的OID
SELECT datname, oid FROM pg_database;

-- 常見的結果：
--     datname     |  oid  
-- ----------------+-------
--  template1      |     1
--  template0      |     0  
--  postgres       | 13067
--  my_database    | 16384
```
**Oracle的檔案結構：**

```bash
# Oracle需要明確的檔案管理
/u01/app/oracle/oradata/ORCLCDB/
├── system01.dbf       # 固定大小，需要管理
├── sysaux01.dbf       # 需要預先規劃大小
├── users01.dbf        # 需要設定擴展策略
└── redo01.log         # 需要設定大小和數量
```

### 5. **效能調校差異**

**Oracle需要DBA精細調校：**

- 要規劃datafile分散到不同磁碟
- 要設定適當的extent size
- 要監控tablespace使用率
- 要手動管理空間增長

**PostgreSQL依賴系統管理員：**

- 使用檔案系統的LVM、RAID等技術
- 依賴檔案系統的快取機制
- 使用檔案系統的壓縮、加密功能
- 透過mount選項調校效能

### 6. **備份恢復差異**

**Oracle：**

```sql
-- Oracle有自己的備份恢復機制
BACKUP TABLESPACE users;
RESTORE TABLESPACE users;
```

**PostgreSQL：**

```bash
# 可以直接使用檔案系統工具
cp -r /var/lib/postgresql/tablespaces/my_data /backup/
rsync -av /var/lib/postgresql/ /backup/
```

## Oracle的資料庫層級管理

### 1. **完整的邏輯存儲架構**

Oracle建立了自己完整的存儲管理體系：

```
Database (資料庫)
    ↓
Tablespace (表格空間)
    ↓
Segment (段) = Table/Index
    ↓ 
Extent (範圍) = 連續的blocks (64K→1M→8M→64M)
    ↓
Block (區塊) = Oracle最小I/O單位 (2K/4K/8K/16K/32K)
    ↓
OS Block (作業系統區塊)
```

### 2. **精細的空間管理控制**

**Block層級管理：**

```sql
-- Oracle自己定義並管理block size
db_block_size = 8192  -- Oracle控制的區塊大小
-- 一個tablespace還可以有非標準block size
CREATE TABLESPACE big_data BLOCKSIZE 32768;
```

**Extent層級管理：**

- Oracle自動管理extent大小：64K → 1M → 8M → 64M
- 可以設定UNIFORM SIZE來固定extent大小
- 追蹤extent的分配與回收

**Segment層級管理：**

```sql
-- Oracle管理segment內部的空間
SEGMENT_SPACE_MANAGEMENT AUTO  -- 使用bitmap管理free space
-- FS1(25%) → FS2(50%) → FS3(75%) → FS4(100%)
```

### 3. **Oracle vs PostgreSQL 的層級差異對比**

|管理層級|Oracle|PostgreSQL|
|---|---|---|
|**最高層級**|資料庫層級自主管理|檔案系統層級依賴|
|**Tablespace**|複雜的datafile管理<br/>SMALLFILE/BIGFILE<br/>AUTOEXTEND設定|簡單的目錄路徑|
|**空間分配**|Extent自動管理<br/>64K→1M→8M→64M|依賴檔案系統分配|
|**Block管理**|Oracle Block (8K)<br/>與OS Block分離|直接使用OS Page|
|**監控工具**|DBA_TABLESPACES<br/>DBA_EXTENTS<br/>DBA_SEGMENTS|檔案系統工具<br/>df, du, ls|

### 4. **具體例子說明差異**

**Oracle的複雜管理：**

```sql
-- Oracle需要管理所有細節
CREATE TABLESPACE sales_data
DATAFILE '/u01/app/oracle/oradata/sales01.dbf' SIZE 100M
AUTOEXTEND ON NEXT 10M MAXSIZE 500M
EXTENT MANAGEMENT LOCAL 
AUTOALLOCATE                    -- Oracle自動管理extent
SEGMENT SPACE MANAGEMENT AUTO   -- Oracle自動管理segment空間
BLOCKSIZE 8192;                 -- Oracle自己的block size
```

**PostgreSQL的簡單方式：**

```sql
-- PostgreSQL交給檔案系統處理
CREATE TABLESPACE sales_data LOCATION '/data/sales';
-- 空間增長：檔案系統自動處理
-- Block大小：使用檔案系統的page size
-- 空間監控：用df, du等工具
```

### 5. **Oracle資料庫層級管理的特點**

**完全自主控制：**

- Oracle有自己的I/O單位(Block)
- Oracle管理space allocation策略
- Oracle追蹤space utilization
- Oracle處理fragmentation

**精細調校能力：**

```sql
-- 可以精確控制每個tablespace的特性
ALTER TABLESPACE users 
ADD DATAFILE '/u01/app/oracle/oradata/users02.dbf' SIZE 200M;

-- 可以調整extent管理策略
ALTER TABLESPACE sales_data 
SHRINK SPACE KEEP 100M;
```

**內建監控機制：**

```sql
-- Oracle提供豐富的空間監控視圖
SELECT tablespace_name, bytes, maxbytes 
FROM dba_data_files;

SELECT segment_name, extents, blocks 
FROM dba_segments;
```

## 總結：兩種管理哲學

**Oracle：資料庫層級管理**

- 優點：精細控制、可預測性能、豐富的調校選項
- 缺點：複雜度高、學習成本大、需要專業DBA

**PostgreSQL：檔案系統層級管理**

- 優點：簡單易用、依賴成熟的檔案系統、容易維護
- 缺點：精細控制較少、依賴OS和檔案系統特性

Oracle選擇在資料庫引擎內部實現完整的存儲管理體系，這讓它能提供enterprise級的存儲控制能力，但也增加了系統的複雜度。這就是為什麼Oracle DBA需要深度了解這些存儲概念，而PostgreSQL管理員更多時候可以依賴檔案系統的管理能力。
## 總結

PostgreSQL的"檔案系統層級管理"意思是：

1. **簡化資料庫層的複雜性**：不像Oracle需要管理extent、datafile大小等
2. **充分利用作業系統功能**：檔案系統的快取、壓縮、加密、RAID等
3. **減少DBA負擔**：很多管理工作交給系統管理員和檔案系統
4. **更好的整合性**：可以使用標準的UNIX/Linux工具進行管理

這種設計哲學讓PostgreSQL更容易部署和維護，但也意味著需要對作業系統有更深的了解，而Oracle則在資料庫層提供了更精細的控制能力。

# 複習與問題
1. [_fsm跟_vm是什麼？](_fsm跟_vm是什麼？.md)
2. [psql怎麼調整block size？](psql怎麼調整block%20size？.md)
3. [怎麼建立tablespace？](怎麼建立tablespace？.md)
