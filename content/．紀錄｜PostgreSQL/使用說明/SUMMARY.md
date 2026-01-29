---
title: 🐘PostgreSQL目錄
maturity: signpost
---
## PostgreSQL
* [1. PostgreSQL介紹](PostgreSQL介紹.md) :版本與可用管理工具
* [2. PostgreSQL安裝](PostgreSQL安裝.md) ：PG安裝步驟
* [3. PostgreSQL管理工具](PostgreSQL管理工具.md) ： pg_、psql、pgAdmin基礎介紹
* [3.1 管理工具PEM安裝](管理工具PEM安裝.md) ：PEM安裝步驟。不符合我的使用情境，可能含有錯誤資訊。
* [4. PostgreSQL連線設定](PostgreSQL連線設定.md) ：postgresql.conf、pg_hba.conf設定
* [5. Postgres Cluster管理](Postgres%20Cluster管理.md) ：建立 PG cluster
	* [5.1 管理資料庫](管理資料庫.md)
	* [5.2 管理Tablespace](管理Tablespace.md)
* [6. Postgres使用者管理](Postgres使用者管理.md) ：增、刪、查、改Role
* [7. Postgres高可用性配置](Postgres高可用性配置.md)：pgpool-2、streaming replication、pgBackRest 簡易說明與架設
* [8. Postgres監控資料庫](Postgres監控資料庫.md)：pg_stat_使用方法、Prometheus+Grafana警報設定
	* [8.1 Prometheus+Grafana下載與設定](Prometheus+Grafana下載與設定.md) 
## pgBackRest

* [1. pgBackRest基礎建置](pgBackRest基礎建置.md)：安裝與初步執行（僅可執行的程度，非正式環境設置）
* [2. pgBackRest基礎還原操作](pgBackRest基礎還原操作.md)：災難復原演練、Contab設定
* [3. pgBackRest參數設定](pgBackRest參數設定.md)：生產環境建議參數
* [4. 備份還原測試自動化](備份還原測試自動化)：不同流量備份還原測試與參數修改
