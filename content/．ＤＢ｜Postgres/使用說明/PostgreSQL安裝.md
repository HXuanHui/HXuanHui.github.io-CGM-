---
theme:
  - default
class: 
paginate: true
header: PostgreSQL安裝
---
<style scoped>h1 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size:60px
}
</style>
<style scoped>p { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size: 16px;
}
</style>
# PostgreSQL安裝
黃暄惠 2024/11

---
## PostgreSQL安裝
 - Linux x86-64
 - Linux x86-32
 - Mac OS X
 - Windows x86-64
 - Windows x86-32(v11後不支援)

---
## 在Windows使用Zip檔安裝

1. 使用zip板下載
   [EDB: Open-Source, Enterprise Postgres Database Management](https://www.enterprisedb.com/download-postgresql-binaries)
1. 解壓縮在你想要放的地方
2. Instance設定詳見「4_PostgreSQL連線設定」

---
## 在Windows使用EDB安裝
1. 選擇適當的版本
   [https://www.postgresql.org/download/windows/](https://www.postgresql.org/download/windows/)
2. 開啟installer
---
## 在Windows使用EDB安裝(cont. 1)
3. 設定資料夾位置
![](．紀錄｜PostgreSQL/picture/image.png)


---
## 在Windows使用EDB安裝(cont. 2)
<style scoped>ul { 
font-size:20px
}
</style>
4. 選擇添加的工具
![width:500px height:400px](．紀錄｜PostgreSQL/picture/chooseTool.png)
- PostgreSQL Server 是資料庫程式
- pgAdmin 4 是 GUI，很好用
- Stack Builder 是管理額外的套件用的
- Command Line Tools 是命令列工具

---

## 在Windows使用EDB安裝(cont. 3)
5. 設定super user postgres密碼
6. 設定port
![](．紀錄｜PostgreSQL/picture/setPSW.png)
7. 一直下一步直到安裝完成

---

## 在Linus安裝(cont. 1)

```bash
# 1. 更新套件列表
sudo apt update

# 2. 安裝PostgreSQL（會安裝最新穩定版本）
sudo apt install postgresql postgresql-contrib

# 3. 檢查服務狀態
sudo systemctl status postgresql

# 4. 啟動並設定開機自動啟動（可選，我沒有做）
sudo systemctl start postgresql
sudo systemctl enable postgresql

# 5. 確認postgresql版本
psql --version
```

---
## 在Linus安裝(cont. 2)
設定環境變數
### 1. 對於postgres用戶

只設定必要的環境變數：

```bash
# 在postgres用戶的.bashrc中
export PGDATA=/var/lib/postgresql/16/main
export PATH=/usr/lib/postgresql/16/bin:$PATH
```

### 2. 對於你的OS用戶

設定完整的環境變數：

```bash
export PGUSER=postgres
export PGHOST=localhost
export PGPORT=5432
export PGDATABASE=postgres
```

---
## 在Linus安裝(cont. 3)
資料庫初始設定

```bash

# 1. 切換到postgres用戶
sudo -i -u postgres

# 2. 進入PostgreSQL命令行
psql

# 3. 設定postgres用戶密碼
ALTER USER postgres PASSWORD 'your_password';

# 4. 創建新的資料庫用戶（可選）
CREATE USER cghadmin WITH PASSWORD 'your_password';
ALTER USER cghadmin CREATEDB;

# 5. 退出psql
\q

# 6. 退出postgres用戶
exit

```
----
## 在Linus安裝(cont. 3)

基本驗證
```bash
# 檢查版本
-u postgres psql -c "SELECT version();"
psql -c "SELECT version();" # 有設定環境變數PGUSER或在postgres user下

# 檢查監聽埠口
netstat -plunt | grep postgres

# 檢查資料庫列表
-u postgres psql -l
psql -l # 有設定環境變數PGUSER或在postgres user下

```

---
## 結語
到目前為止（windows）完成下載
 - PostgreSQL Server 
 - pgAdmin 4 是 GUI
 - Command Line Tools(psql)
 - Stack Builder (有選擇下載才有，zip檔沒有)

可以在server利用psql連線，遠端連線還須設定。


---
# 複習與問題
1. [管理工具PEM安裝](管理工具PEM安裝.md)