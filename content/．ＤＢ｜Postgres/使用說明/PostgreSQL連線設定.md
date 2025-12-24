---
theme:
  - default
class: 
paginate: true
header: PostgreSQL連線設定
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

# PostgreSQL連線設定
黃暄惠 2024/11

---
## 工具與文件

- pg_hba.conf
- postgresql.conf
- psql

⭐PostgreSQL沒有TNSNAME設定（不透過listener綁定port）

---

## Postgres連線設定-postgresql.conf

在 postgresql.conf 檔案中變更監聽位址。

依預設，PostgreSQL 允許監聽 localhost 連接，不允許遠端 TCP/IP 連接。

可以打入一到多個ip位址，以逗號做分隔，或是輸入'米字號'代表所有的ip皆可連線

![](．紀錄｜PostgreSQL/picture/conf.png)

---
<style scoped> 
<style scoped> table { width: 100%; /* 设置表格宽度 */ border-collapse: collapse; /* 合并边框 */ } th, td { text-align: left; /* 左对齐 */  border: none; font-size:22px } th:nth-child(1), td:nth-child(1) { width: 55%; /* 左边列宽度为60% */ } th:nth-child(2), td:nth-child(2) { width: 45%; /* 中间列宽度为20% */ } </style>
## Postgres連線設定-pg_hba.conf

檔案裡面已經有數條內建的規則，每一條規則都是由五個欄位組成，以下簡單說明各個欄位的用途：

| ![width:900px height:400px](．紀錄｜PostgreSQL/picture/pg_hba.png) | TYPE：連線的方式。ex. local, host<br>DATABASE：指定適用該筆規則的資料庫。<br>USER：規則套用的使用者或者使用者群組’(’前面加上加號+。<br>ADDRESS：要套用的來源IP。ex. IPV4或IPV6。<br>METHOD：接受的驗證方法。ex. trust, peer, md5/scram-sha-256 |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |



---
## 連線-PSQL command line

psql -h (host IP) -p (port) -U (Username) -d (DB name) -W(password)

```powershell
psql -h 10.30.111.50 -p 5432 -U postgres -d lnka1
```
或
```powershell
psql postgresql://userName:password@localhost/mydatabase
```

---
## 結語
到這裡連接上資料庫就可以做簡單的資料庫操作了，前提是要先用pg_ctl啟動資料庫，詳情請見5_PostgreSQL Instance管理。

---
# 複習與問題
1. [PG是怎麼綁port的？跟ORCL的差異是什麼？](PG是怎麼綁port的？跟ORCL的差異是什麼？.md)
2. [怎麼在psql切換DB？](怎麼在psql切換DB？.md)
3. [user可以連縣adb，也可以連線bdb，但是連入adb之後不可以＼c切換至bdb，是不是辦不到？](user可以連縣adb，也可以連線bdb，但是連入adb之後不可以＼c切換至bdb，是不是辦不到？.md)
4. [PG可以跟ORCL一樣設置多PORT來達到附載平衡嗎？](PG可以跟ORCL一樣設置多PORT來達到附載平衡嗎？.md)


---
## Reference

1. [Postgres連線設定](https://ithelp.ithome.com.tw/articles/10287261)
2. [16免安裝版-架設資料庫server與client端使用筆記](https://medium.com/jimmy-wang/postgresql-16%E5%85%8D%E5%AE%89%E8%A3%9D%E7%89%88-%E6%9E%B6%E8%A8%AD%E8%B3%87%E6%96%99%E5%BA%ABserver%E8%88%87client%E7%AB%AF%E4%BD%BF%E7%94%A8%E7%AD%86%E8%A8%98-%E5%90%AB%E5%AE%89%E8%A3%9D%E5%90%91%E9%87%8F%E8%B3%87%E6%96%99%E6%93%B4%E5%85%85pgvector-9f1100738ad1)
3. [連接遠端 Remote PostgreSQL 資料庫](https://www.netiq.com/zh-tw/documentation/identity-manager-47/setup_windows/data/t45mzvjb68l4.html)
4. [PostgreSQL 連線安全設定指南](https://ianwu.tw/press/postgresql/tutorial/security_connection.html#google_vignette)
5. [設定外網可以連結本地端postgresql資料庫](https://medium.com/@abcd22774403/%E8%A8%AD%E5%AE%9A%E5%A4%96%E7%B6%B2%E5%8F%AF%E4%BB%A5%E9%80%A3%E7%B5%90%E6%9C%AC%E5%9C%B0%E7%AB%AFpostgresql%E8%B3%87%E6%96%99%E5%BA%AB-bedcdbd8f93e)