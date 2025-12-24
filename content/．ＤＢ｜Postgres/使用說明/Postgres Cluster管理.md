---
theme:
  - default
class: []
paginate: true
header: Postgres Instance管理
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



# Postgres Cluster管理
黃暄惠 2024/11

---
## 叢集的啟動與管理

💡PostgreSQL的啟動過程相對簡單，只要**啟動叢集**就等於**開啟資料庫**。 當你啟動PostgreSQL時，叢集會直接進入運行狀態，並且會自動加載所有必要的數據結構和配置，而不需要經歷像Oracle那樣的多個狀態階段。



---

## 初始化新的PostgreSQL 


到安裝檔路徑下bin資料夾，開啟終端初始化，所以要創新叢集（以orcl來說就是Instance）的時候就運行來複製一個初始的叢集，在利用conf來控制參數。

```powershell
.\initdb.exe -D ..\data --username=postgres --auth=trust
```

![](．紀錄｜PostgreSQL/picture/init.png)


---
## 初始化新的PostgreSQL-參數設定
以下參數只能在初始化或初始化前決定，無法事後透過conf檔或sql修改。
以下資訊也可以透過PgAdmin4資料庫儀錶板-配置取得。
```sql

select name, setting,unit,context,vartype,source from pg_settings order by name;
```

| 參數名稱                             | 設定值                                 | 單位  | Context  | 類型       | 來源      | 決定時機      |
| -------------------------------- | ----------------------------------- | --- | -------- | -------- | ------- | --------- |
| block_size                       | 8192                                |     | internal | integer  | default | 編譯時決定     |
| data_checksums                   | off                                 |     | internal | bool     | default | initdb時決定 |
| data_directory_mode              | 0700                                |     | internal | integer  | default | initdb時決定 |
| debug_assertions                 | off                                 |     | internal | bool     | default | 編譯時決定     |
| in_hot_standby                   | off                                 |     | internal | bool     | default | 運行時狀態     |
| integer_datetimes                | on                                  |     | internal | bool     | default | 編譯時決定     |
| max_function_args                | 100                                 |     | internal | integer  | default | 編譯時決定     |
| max_identifier_length            | 63                                  |     | internal | integer  | default | 編譯時決定     |
| max_index_keys                   | 32                                  |     | internal | integer  | default | 編譯時決定     |
| segment_size                     | 131072                              | 8kB | internal | integer  | default | 編譯時決定     |
| server_encoding                  | UTF8                                |     | internal | string   | default | initdb時決定 |
| server_version                   | 16.9 (Ubuntu 16.9-0ubuntu0.24.04.1) |     | internal | string   | default | 編譯時決定     |
| server_version_num               | 160009                              |     | internal | integer  | default | 編譯時決定     |
| shared_memory_size               | 143                                 | MB  | internal | integer  | default | 運行時計算     |
| shared_memory_size_in_huge_pages | 72                                  |     | internal | integer  | default | 運行時計算     |
| ssl_library                      | OpenSSL                             |     |          | internal | string  | default   |
| wal_block_size                   | 8192                                |     | internal | integer  | default | 編譯時決定     |
| wal_segment_size                 | 16777216                            | B   | internal | integer  | default | initdb時決定 |

---

## 初始化新的PostgreSQL-參數設定（cont.）

🚨block_size必須在postgresql編譯前就修改！！！

---
## 初始化新的PostgreSQL-參數設定（cont.）

在pg_settings中：
 - internal：編譯時或initdb時決定
 - postmaster：需要重啟才能修改
 - sighup：可以重新載入設定
 - superuser-backend：超級用戶可修改（新連線生效）
 - backend：每個連線可修改
 - user:一般用戶可修改


---
## 啟動PostgreSQL叢集

**🚨PostgreSQL Instance的啟動與停止在windows上只能使用pg_ctl，無法藉由其他工具。**

```powershell
.\pg_ctl -D D:\postgresql-版本號-windows-x64-binaries\pgsql\data -l logfile start
```


![](．紀錄｜PostgreSQL/picture/start.png)
若有設定環境參數，則可省略pg_ctl的位置及-D "{datadir}"。
```powershell
pg_ctl start
```


---
## 測試服務狀態
這個指令會顯示資料庫的當前狀態。

```powershell
.\pg_ctl -D D:\postgresql-版本號-windows-x64-binaries\pgsql\data status
```


---

## 關閉PostgreSQL叢集

```powershell
.\pg_ctl -D D:\postgresql-版本號-windows-x64-binaries\pgsql\data stop
```

這個指令會使用預設的 “fast” 模式來關閉資料庫。你也可以指定不同的關閉模式：

- **-m smart**：等待所有連線斷開後再關閉。
- **-m fast**：（預設）立即斷開所有連線並關閉。
- **-m immediate**：立即關閉，不等待任何操作完成（可能導致資料損壞）。
---

## 重載PostgreSQL叢集

對於'sighup'的參數可以用重載的方式重新讀入。

- **使用 `pg_reload_conf()` SQL 命令**:
	```powershell
	# 已使用psql連線來重讀設定檔
	SELECT pg_reload_conf();
	```
	
- **使用 `pg_ctl` 命令**:
	```powershell
	# 於cmd透過pg_ctl指定DB server重讀設定檔
	.\pg_ctl -D D:\postgresql-版本號-windows-x64-binaries\pgsql\data reload 
    ```

---
## 重啟PostgreSQL叢集

對於'postmaster'的參數必須用重啟的方式重新讀入。

```powershell
.\pg_ctl -D D:\postgresql-版本號-windows-x64-binaries\pgsql\data restart [-m smart|fast|immediate]
```

---
# 複習與問題
1. [psql怎麼調整block size？](psql怎麼調整block%20size？.md)
2. [ALTER SYSTEM 可以改多少參數？](ALTER%20SYSTEM%20可以改多少參數？.md)
3. [psql instance關閉怎麼做？systemctl 跟 pg_ctl的差別是什麼？](psql%20instance關閉怎麼做？systemctl%20跟%20pg_ctl的差別是什麼？.md)

---
<style scoped>h2 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
}
p{
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size:24px
}
</style>
## PostgreSQL Instance架構概述

後面是架構科普，操作看前面就好🙂

---


**叢集（Instance）**：
在PostgreSQL中，叢集是一個運行中的PostgreSQL伺服器，它管理著一組資料庫。每個叢集可以同時服務多個資料庫，這資料庫之間是完全獨立的。


![bg left 100%](．紀錄｜PostgreSQL/picture/aa2b6aa64538148c24beb678f0506807.png)


---

**資料庫（Database）**： 
資料庫之間不共享資料，並且每個資料庫都有自己的綱要（Schema）和對象，如表、索引和視圖等。

![bg left 100%](．紀錄｜PostgreSQL/picture/aa2b6aa64538148c24beb678f0506807.png)

---

**綱要（Schema）**： 
在每個資料庫內部，可以有多個綱要，通常會有一個名為`public`的默認綱要。用戶可以在不同的綱要中創建表和其他對象，以便更好地管理和隔離數據。

![bg left 100%](．紀錄｜PostgreSQL/picture/aa2b6aa64538148c24beb678f0506807.png)


--- 
### Instance啟動過程打開文件順序
也不是很重要因為Cluster的開啟階段並不像ORACLE一樣複雜。

1. `global/pg_control`
2. `pg_notify/0000`
3. `./postgresql.conf`
4. `./postmaster.opts`
5. `./pg_xlog/00000001` 現在是叫pg_wal
6. `./pg_hba.conf`
7. `./global/1137 pg_pltemplate_name_index`(接下來就是遍歷global文件…)


---

# 複習與問題
1. [PG是怎麼綁port的？跟ORCL的差異是什麼？](PG是怎麼綁port的？跟ORCL的差異是什麼？.md)
2. [psql的資料分布？](psql的資料分布？.md)

---
### Reference

1. [pg_ctl | PostgreSQL 正體中文使用手冊](https://docs.postgresql.tw/reference/server-applications/pg_ctl)
2. [PostgreSQL数据库初始化](https://leapbook.readthedocs.io/en/latest/postgresql/database_postgresql_create_database.html#id5)
3. [PostgreSQL: Documentation: 15: Chapter 20. Server Configuration](https://www.postgresql.org/docs/15/runtime-config.html)
4. [PostgreSQL的結構](https://blog.csdn.net/penriver/article/details/119680114)