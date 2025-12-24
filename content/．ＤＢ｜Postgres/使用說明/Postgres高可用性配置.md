---
theme:
  - default
class: 
paginate: true
header: Postgres高可用性配置
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

# Postgres高可用性配置
黃暄惠 2024/11

---

## 方案一工具
以下工具可以建構資料庫雙機熱備與災備架構，建議建立在CentOS/Rocky Linux/RHEL上

 - 💰failover manager：故障轉移

 - pgpool-2：負載平衡⭐要禁用故障轉移

 - streaming replication：Active-Passive架構資料同步

 - 💰barman：資料備份還原

---

## 方案二工具
不使用企業工具的簡易版HA架構，建議建立在Rocky Linux/RHEL上
 - pgpool-2：故障轉移、負載平衡
 - streaming replication：Active-Passive架構資料同步
 - pgBackRest：資料備份還原 

---
<style scoped>h2 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size:60px
}
</style>
## 建立HA架構

- **伺服器需求**：準備至少兩台伺服器作為 PostgreSQL 的主從節點。  

---
<style >h3 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
}
</style>

### 設定 Streaming Replication

**主節點設定**：
1. 編輯 `postgresql.conf` 文件，啟用以下參數：
     ```plaintext
     archive_mode = on
	 max_wal_senders = 5
	 max_wal_size = 10GB
	 wal_level = replica
	 hot_standby = on
	 archive_command = 'rsync -a %p /opt/pg_archives/%f'
     ```
     
2. 編輯 `pg_hba.conf`，允許從節點連接：
     ```plaintext
     host replication all <standby_ip>/32 trust
     ```

3. 重啟資料庫

---
### 設定 Streaming Replication (Cont. 2)

⭐此示範環境為Windows
**從節點設定**：
1. 初始化一個instance，並且刪除所有資料夾中的檔案
	```powershell
	.\initdb.exe -D "..\stb-db" --username=postgres --auth=trust
	Remove-Item "..\stb-db\*" -Recurse -Force
	```
1. basebackup主伺服器
	```powershell
	.\pg_basebackup -h 127.0.0.1 -p 5432 -U replicator -D "..\stb-db" 
	-P --wal-method=stream --write-recovery-conf
	```

---

### 設定 Streaming Replication (Cont. 3)

**從節點設定**：
3. 編輯 `postgresql.conf` 文件
```plaintext
restore_command = 'rsync -a  postgres@192.168.57.101:/opt/pg  
_archives/%f %p' 
recovery_target_timeline = 'latest'
```

---

### 設定 Streaming Replication (Cont. 4)
**檢查複製狀態**：在主節點上執行以下命令確認複製狀態：
  ```sql
  SELECT * FROM pg_stat_replication;
  ```
![](．紀錄｜PostgreSQL/picture/螢幕擷取畫面%202024-10-30%20171329.png)


---

### 安裝和配置 pgpool-II

- **安裝 pgpool-II**：
  在中介伺服器上安裝 `pgpool-II`。
  
- **配置 pgpool-II**：
  編輯 `pgpool.conf` 文件，設定主從節點的連接資訊和負載平衡參數。

	```plaintext
	backend_hostname0 = '<primary_ip>'
	backend_port0 = '<primary_port>'
	backend_weight0 = 1
	
	backend_hostname1 = '<standby_ip>'
	backend_port1 = '<standby_port>'
	backend_weight1 = 1
	```

---

### 安裝和配置 pgpool-II(Cont. 2)

 - **啟用故障轉移功能：**
    ```plaintext
    health_check_period = 10 
    health_check_timeout = 20 
    health_check_user = 'postgres'
    ```

---

### 資料備份還原 pgBackRest

- **安裝 pgBackRest**
- **配置 pgBackRest**
	```plaintext
	[demo] 
	pg1-path=/var/lib/postgresql/14/main
	 
	[global] 
	repo1-path=/var/lib/pgbackrest repo1-retention-full=2 
	
	[global:archive-push] 
	archive_command='pgbackrest --stanza=demo archive-push %p'	
	```

---

### 資料備份還原 pgBackRest(Cont. 2)

⭐此示範環境為Unix-like OS
- **執行完整備份**
	```bash
	sudo -u postgres pgbackrest stanza-create --stanza=demo
	```

 - **還原備份**
	 ```bash
	 sudo systemctl stop postgresql
	 sudo -u postgres pgbackrest --stanza=demo restore
	 sudo systemctl start postgresql
	 ```

---

# 複習與問題
1. [當機復原機制是甚麼？](當機復原機制是甚麼？.md)
2. [pgBackRest基礎建置](pgBackRest基礎建置.md)
---

### Reference
1. [Streaming Replication: A Step-By-Step Setup Guide | EDB](https://www.enterprisedb.com/blog/how-set-streaming-replication-keep-your-postgresql-database-performant-and-date)
2. [pgBackRest - Reliable PostgreSQL Backup & Restore](https://pgbackrest.org/)
3. [EDB Docs - Installation](https://www.enterprisedb.com/docs/supported-open-source/pgbackrest/02-installation/)
4. [EDB 對 Pgpool II 的使用建議](https://www.enterprisedb.com/postgres-tutorials/edbs-recommendation-pgpool-ii-usage)
