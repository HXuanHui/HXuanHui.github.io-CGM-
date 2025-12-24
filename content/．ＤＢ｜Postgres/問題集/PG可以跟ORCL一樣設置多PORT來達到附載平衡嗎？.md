#postgreSQL 
PostgreSQL**可以**設置多個埠號，但運作方式和Oracle有本質差異。讓我詳細說明：

## PostgreSQL多埠號設置

### 基本配置方法

**方法1：單一實例監聽多個埠號**

```bash
# postgresql.conf
listen_addresses = '*'
port = 5432,5433,5434  # ❌ 這樣設置是錯誤的！PostgreSQL不支援這種語法
```

**正確的方法：需要啟動多個PostgreSQL實例**

```bash
# 建立多個資料目錄
$ initdb -D /var/lib/postgresql/data1
$ initdb -D /var/lib/postgresql/data2
$ initdb -D /var/lib/postgresql/data3

# 配置不同的埠號
# /var/lib/postgresql/data1/postgresql.conf
port = 5432

# /var/lib/postgresql/data2/postgresql.conf  
port = 5433

# /var/lib/postgresql/data3/postgresql.conf
port = 5434

# 啟動多個實例
$ pg_ctl -D /var/lib/postgresql/data1 start
$ pg_ctl -D /var/lib/postgresql/data2 start  
$ pg_ctl -D /var/lib/postgresql/data3 start
```

## 與Oracle的核心差異

### Oracle的優勢：單一資料庫實例

**Oracle架構：**

```
單一資料庫實例 (ORCLCDB)
    ↓
多個Listener程序 (LISTENER, LISTENER2)
    ↓  
監聽不同埠號 (1521, 1522)
    ↓
共享相同的資料和記憶體結構
```

**實際效果：**

```bash
# Oracle可以這樣做
$ lsnrctl start LISTENER   # 監聽1521
$ lsnrctl start LISTENER2  # 監聽1522

# 兩個listener都服務同一個資料庫
$ sqlplus hr/pass@host:1521/ORCLCDB  # 相同的資料
$ sqlplus hr/pass@host:1522/ORCLCDB  # 相同的資料
```

### PostgreSQL的限制：

**PostgreSQL架構：**

```
多個PostgreSQL實例
    ↓
實例1(5432) | 實例2(5433) | 實例3(5434)
    ↓            ↓            ↓
獨立的資料   獨立的資料   獨立的資料
獨立的記憶體 獨立的記憶體 獨立的記憶體
```

**問題：**

```bash
# PostgreSQL會是這樣
$ psql -p 5432 -d mydb  # 連到實例1的資料
$ psql -p 5433 -d mydb  # 連到實例2的資料 (可能完全不同！)
```

## PostgreSQL實現負載平衡的正確方法

### 1. 使用連線池 (推薦方案)

**PgBouncer配置：**

```bash
# /etc/pgbouncer/pgbouncer.ini
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = scram-sha-256
pool_mode = transaction
max_client_conn = 1000      # 最多1000個客戶端連線
default_pool_size = 20      # 實際到PostgreSQL只有20個連線

# 啟動PgBouncer
$ pgbouncer /etc/pgbouncer/pgbouncer.ini

# 客戶端連線到連線池
$ psql -h localhost -p 6432 -U username -d mydb
```

**效果類似Oracle的共享伺服器：**

```
1000個客戶端連線 → PgBouncer → 20個PostgreSQL連線
```

### 2. 使用HAProxy進行負載平衡

**配置多個PostgreSQL讀副本：**

```bash
# haproxy.cfg
backend postgres_read
    balance roundrobin
    server pg1 192.168.1.10:5432 check
    server pg2 192.168.1.11:5432 check
    server pg3 192.168.1.12:5432 check

frontend postgres_frontend
    bind *:5432
    default_backend postgres_read
```

### 3. PostgreSQL內建的並行處理

**PostgreSQL的內建負載分散：**

```sql
-- PostgreSQL會自動使用多個worker程序
SET max_parallel_workers_per_gather = 4;

-- 複雜查詢會自動並行化
EXPLAIN (ANALYZE, BUFFERS) 
SELECT COUNT(*) FROM large_table WHERE condition;
```

## 實際效能比較

### Oracle多Listener方案

**優點：**

```bash
# 真正的負載分散，共享相同資料
$ netstat -tulpn | grep 1521
tcp 0 0 0.0.0.0:1521 LISTEN  # LISTENER
$ netstat -tulpn | grep 1522  
tcp 0 0 0.0.0.0:1522 LISTEN  # LISTENER2

# 連線可以分散到不同的Listener程序
# 但存取相同的SGA和資料檔案
```

**效能監控：**

```sql
-- 查看不同Listener的負載
SQL> SELECT program, count(*) FROM v$session 
     WHERE type = 'USER' GROUP BY program;

PROGRAM                    COUNT(*)
------------------------- ---------
sqlplus@client1 (TNS V1-V3)    50    -- 透過LISTENER
sqlplus@client2 (TNS V1-V3)    45    -- 透過LISTENER2
```

### PostgreSQL連線池方案

**PgBouncer效能：**

```bash
# 監控PgBouncer狀態
$ psql -p 6432 -U pgbouncer -d pgbouncer -c "SHOW POOLS;"

database  | user      | cl_active | cl_waiting | sv_active | sv_idle
----------|-----------|-----------|------------|-----------|--------
mydb      | myuser    |       150 |         20 |        18 |       2

# cl_active: 活躍的客戶端連線 (150)
# sv_active: 活躍的伺服器連線 (18) ← 大幅減少！
```

## 最佳實務建議

### 對於PostgreSQL環境

**小型到中型應用：**

```bash
# 使用PgBouncer就足夠了
Application → PgBouncer → Single PostgreSQL Instance
```

**大型高可用環境：**

```bash
# 使用讀寫分離 + 連線池
Write Operations → PgBouncer → Master PostgreSQL
Read Operations  → HAProxy → Multiple Read Replicas
```

**企業級部署：**

```bash
# 結合多種技術
Applications → PgBouncer → HAProxy → PostgreSQL Cluster
           (連線池)    (負載平衡)     (高可用性)
```

### 配置範例：完整的PostgreSQL負載平衡方案

```bash
# 1. 設定PostgreSQL串流複製
# Master配置
echo "wal_level = replica" >> /etc/postgresql/postgresql.conf
echo "max_wal_senders = 3" >> /etc/postgresql/postgresql.conf

# 2. 配置PgBouncer
cat > /etc/pgbouncer/pgbouncer.ini << EOF
[databases]
* = host=localhost port=5432

[pgbouncer]
listen_port = 6432
pool_mode = transaction
max_client_conn = 500
default_pool_size = 25
EOF

# 3. 配置HAProxy (如果有多個副本)
cat > /etc/haproxy/haproxy.cfg << EOF
backend postgres_read
    balance roundrobin
    server pg1 127.0.0.1:6432 check
    server pg2 127.0.0.1:6433 check
EOF
```

## 總結對比

|特性|Oracle多Listener|PostgreSQL方案|
|---|---|---|
|**設定難易度**|中等|容易（PgBouncer）|
|**資源使用**|相同資料，多個程序|需要額外中介軟體|
|**真正負載分散**|✅ 是|✅ 是（透過連線池）|
|**高可用性**|❌ 單點故障|✅ 可配置多副本|
|**維護複雜度**|低|中等|

**結論：**PostgreSQL無法像Oracle那樣簡單地用多埠號實現負載平衡，但透過PgBouncer + HAProxy的組合，可以達到更好的效能和可用性。Oracle的多Listener方案簡單但不提供真正的高可用性，而PostgreSQL的方案雖然複雜一些，但更符合現代分散式架構的需求。