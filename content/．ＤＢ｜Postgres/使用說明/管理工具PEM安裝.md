---
theme:
  - default
class: []
paginate: true
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

# PEM安裝
黃暄惠 2024/11

---

在 [EnterpriseDB 網站上](https://www.enterprisedb.com/products/postgres-enterprise-manager)創建一個帳戶。

在安裝[頁面](https://www.enterprisedb.com/software-downloads-postgres)找到PEM

![width:600px height:450px](．紀錄｜PostgreSQL/picture/{1E3BADB7-E8C4-4895-908B-2B4CB8B059DB}.png)

---
<style>h2 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
}
</style>
<style scoped>p { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
}
</style>
## 下載Agnet

負責收集和傳送監控數據，安裝於多個需要被監管的伺服器上。

---

1. 右鍵按單擊下載的文件，然後選擇 **Run as administrator（以管理員身份運行**）以啟動安裝。

	將出現安裝精靈。點擊下一步繼續。

	![](．紀錄｜PostgreSQL/picture/image.jfif)

---

2. 接受許可協議，然後按下一步繼續。


	![](．紀錄｜PostgreSQL/picture/{D0B56362-F191-4D4C-8805-55827E66775E}.png)

---

3. 選擇安裝目錄或保留預設設置，然後按下一步

	![](．紀錄｜PostgreSQL/picture/{8ADB264A-3640-469E-8711-1D1AE8B83C02}.png)


---

4. 提供 PEM 代理的密碼（系統使用者的密碼），然後下一步。

	![](．紀錄｜PostgreSQL/picture/{584C4E29-4BAB-4EF9-8432-E388215A0252}.png)

---
5. 完成
	![](．紀錄｜PostgreSQL/picture/{E61610F2-3B0A-402B-B600-229086819E6F}.png)
---
<style scoped>
p { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
}
</style>
## 下載Server

包含 PostgreSQL 實例和 Apache 網頁伺服器，提供網頁介面，安裝於管理人員使用的伺服器/電腦上。

---

1. 接受許可協議，然後按下一步繼續。

	![](．紀錄｜PostgreSQL/picture/{D0B56362-F191-4D4C-8805-55827E66775E}.png)

---
2. 選擇安裝目錄或保留預設設置，然後按下一步

	![](．紀錄｜PostgreSQL/picture/{8ADB264A-3640-469E-8711-1D1AE8B83C02}.png)

---
3. 選擇要下載的服務，由於我使用開源PostgreSQL，就不下載EDB PostgreSQL Server了。
	![](．紀錄｜PostgreSQL/picture/{6C082706-E237-437C-AEEB-5DE20D4F1889}.png)

---

4. 還需要繼續下載httpd
	![](．紀錄｜PostgreSQL/picture/{DE203F4B-081F-48A8-870D-C4F94D1F533E}.png)


---

**安裝httpd**

1. 選擇安裝目錄或保留預設設置，然後按下一步
	![](．紀錄｜PostgreSQL/picture/{7519A1B6-D798-4CBF-8D73-232EF44BAF2E}.png)


---

2. 為 Apache 配置 port（可以保留預設值），然後按下 「Next」
	![](．紀錄｜PostgreSQL/picture/{4624F691-8E8D-47F3-972B-4DAC668A5348}.png)

---
3. 設置管理server
	如右圖，管理server中會有名為pem的database，第一次建立會自動設置。

| ![](．紀錄｜PostgreSQL/picture/{F08AAD25-9DC7-497C-A2CD-A7604BFE0963}.png) | ![](．紀錄｜PostgreSQL/picture/{E612001C-CB1E-4AE8-B55E-AF337746F277}.png) |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------- |


---

![](．紀錄｜PostgreSQL/picture/{8560ED30-16CE-42C4-A6F7-437EF6B5C000}.png)

---

開啟Web Client

URL：[https//ip_address_of_pem_host：8443/pem](https://ip_address_of_pem_host:8443/pem)
![width:500px height:400px](．紀錄｜PostgreSQL/picture/{27A681E8-4BB3-43F2-B710-D5725ADE2A5B}.png)

---

**References**
1. [https://www.enterprisedb.com/docs/pem/latest/installing/](https://www.enterprisedb.com/docs/pem/latest/installing/)
2. [https://www.enterprisedb.com/docs/pem/latest/installing/windows/](https://www.enterprisedb.com/docs/pem/latest/installing/windows/)
3. [有關安裝適用於 Windows 的 Postgres Enterprise Manager 的分步指南。- DEV 社區](https://dev.to/chidera/step-by-step-guide-on-installing-postgres-enterprise-manager-for-windows-3j4c)
4. [https://www.enterprisedb.com/docs/pem/latest/](https://www.enterprisedb.com/docs/pem/latest/)
