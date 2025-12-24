---
theme:
  - default
class: 
paginate: true
header: Postgres監控資料庫
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
}
</style>
# Postgres監控資料庫
黃暄惠 2024/11

---
<style> h2 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
}
</style>
## Postgres監控資料庫工具

1. Postgres插件
2. PEM
4. Prometheus+Grafana

可用工具但不在此文件中：
- pgCluu
- pgBadger

---

## Postgres插件使用方法


---
### pg_stat_statements

**用途:系統執行過的SQL指令與執行情況**
⭐有些插件在使用前必須先加到postgresql.conf的shared_preload_libraries或是session_preload_libraries裡面，這樣插件才能跟隨伺服器/session啟動。
⭐僅有超級用戶或pg_read_all_stats 角色成員可以查看 `pg_stat_statements` 的詳細資訊。

```sql
CREATE EXTENSION pg_stat_statements;
SELECT * FROM pg_stat_statements;
```

![](．紀錄｜PostgreSQL/picture/pg_stat.png)


---
### pg_stat_user_indexs

**用途:查詢資料庫index的使用情形，可進一步優化索引使用**

```sql
SELECT * FROM pg_stat_user_indexes;

-- 查看特定表的索引統計
SELECT * FROM pg_stat_user_indexes WHERE relname = 'your_table_name';

-- 查找未使用的索引
SELECT 
	schemaname, relname AS tablename,
	indexrelname AS indexname,
	idx_scan AS index_scans 
FROM 
	pg_stat_user_indexes 
WHERE 
	idx_scan = 0;
```



---
### pg_stat_activities

**用途:可以查看Postgres主機現在正在執行的每個backend process以及其資訊**

```sql
SELECT * FROM pg_stat_activity;

-- 查詢閒置連接
SELECT COUNT(*) FROM pg_stat_activity WHERE state = 'idle';

-- 查找未完成的事務
SELECT * FROM pg_stat_activity WHERE xact_start IS NOT NULL AND state != 'idle';
```

![](．紀錄｜PostgreSQL/picture/stat_activities.png)


---
### pg_stat_database

**用途:可以查看Postgres主機上面每個資料庫被使用的情形，例如：commit/中斷的交易數量，被讀取/插入/更新/刪除的tuple數量等等**


```sql
SELECT * FROM pg_stat_database;

-- 分析查詢性能
SELECT 
	datname, 
	blks_read, 
	blks_hit, 
	(blks_hit::float / (blks_hit + blks_read)) * 100 AS hit_ratio 
FROM 
	pg_stat_database;
```

![](．紀錄｜PostgreSQL/picture/stat_database.png)


---
### 其他
更多統計資料及用法可以參考[官網文件 The Cumulative Statistics System](https://www.postgresql.org/docs/current/monitoring-stats.html)

---
## PEM設定警報的步驟

1. **進入管理介面**：
   - 登入 PostgreSQL Enterprise Manager，從主選單選擇「管理」(Management) 標籤。

2. **管理警報**：
   - 在管理選單中，點擊「管理警報」(Manage Alerts) 選項。


---

3. **創建新警報**：
   - 點擊「新增」(Plus) 按鈕來設置新的警報。這裡會顯示多種可用的預設警報選項。
![](．紀錄｜PostgreSQL/picture/add_alert.png)
---

4. **選擇警報類型**：
   - 從可用的警報列表中選擇你需要的類型，例如「自動清理警報」(Auto Vacuum Alert)。選擇後，輸入相應的閾值（threshold values），然後保存這個警報。
![width:500px height:400px](．紀錄｜PostgreSQL/picture/alert_setting.png)

---

5. **設定自動創建**：
   - 如果希望對新添加的對象（如代理或伺服器）自動創建警報，可以在創建警報模板時勾選「自動創建」(Auto Create) 選項，並設定必要的閾值。這樣當新對象被添加時，系統會自動生成相應的警報。
![](．紀錄｜PostgreSQL/picture/alert_auto.png)


---

## Prometheus & Grafana創建警報

1. 在左側導航欄中，選擇 **Alerting** -> **Alert Rules**。
![bg right 50%](．紀錄｜PostgreSQL/picture/grafana_add_alert.png)




---

2. 在 **Alerting** 標籤下，點擊 **Create Alert**。
2. 設定評估條件，例如：
    
	- 在 **Query** 標籤中，選擇您的數據源並配置查詢語句。
	- 在 **Conditions** 部分，設定告警的觸發條件，例如使用 `avg()` 函數來檢查某個指標的平均值是否超過特定閾值。
	- 設定 **Evaluate every** 和 **For**，例如每分鐘檢查一次，並且在超過閾值後持續超過五分鐘才觸發告警。
	- 設定 **Rule Name**（規則名稱）、**Folder**（分類資料夾）及 **Group**（群組）。
	- 可選擇添加摘要和註解，以提供更多上下文信息
    
4. 添加條件，例如當某個指標超過閾值時觸發警報。


---
 **設定通知通道**

1. 在左側導航欄中，選擇 **Alerting** -> **Contact Points**，然後點擊 **+ New contact point**。
	 - 選擇通知類型，例如 Email、Slack 或 Discord 等，並填入相應的 webhook 地址或其他必要信息
1. 測試通知通道以確保配置正確。

---
## Reference

1. [PostgreSQL: Documentation: 17: 27.2. The Cumulative Statistics System](https://www.postgresql.org/docs/current/monitoring-stats.html)
2. [Grafana | How to Setup Alert in Grafana ](https://blog.bimap.com.tw/2023/03/29/grafana-alert-setting)
3. [pgCluu下載與設定](pgCluu下載與設定.md)
4. [pgBadger下載與設定](pgBadger下載與設定.md)

---
### 相關文件
1. Prometheus+Grafana下載與設定