---
theme:
  - default
paginate: true
class: []
header: PostgreSQL介紹
footer:
---
<style scoped> h1 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size: 60px;
}</style>
<style scoped>p { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size: 16px;
}
</style>
# PostgreSQL介紹
黃暄惠 2024/11

---

## 主要特點

- **開源與免費**
- **高度可擴展性**：
- **豐富的資料類型**：包括 JSON、XML、陣列、幾何圖元等，並允許使用者自定義資料類型。
- **事務完整性**：支援 ACID 特性（原子性、一致性、隔離性、持久性），以及預寫日誌（WAL）技術，確保資料的安全與一致性。
- **可程式化性**：使用者可以透過多種程式語言（如 PL/pgSQL、PL/Python 等）編寫函式和觸發器，以滿足特定需求。

---


---
<style scoped> table {
width: 90%; /* 设置此幻灯片的表格宽度为50% */
font-size: 20px; /* 设置字体大小 */ 
} </style>
## 版本
每年的9月會發布新的主要版本，每三個月發布次要版本（錯誤修復）。 每個版本將在首次發布之後第5年11月EOS。

| **Version** | **Current minor** | **Supported** | **First Release**  | **Final Release** |
| ----------- | ----------------- | ------------- | ------------------ | ----------------- |
| 17          | 17.0              | Yes           | September 26, 2024 | November 8, 2029  |
| 16          | 16.4              | Yes           | September 14, 2023 | November 9, 2028  |
| 15          | 15.8              | Yes           | October 13, 2022   | November 11, 2027 |
| 14          | 14.13             | Yes           | September 30, 2021 | November 12, 2026 |
| 13          | 13.16             | Yes           | September 24, 2020 | November 13, 2025 |
| 12          | 12.20             | Yes           | October 3, 2019    | November 14, 2024 |
| 11          | 11.22             | No            | October 18, 2018   | November 9, 2023  |


---
### 用戶端與伺服器端

- 官方沒有提供**compatibility matrix**
- 除了功能不同外（syntax等），其餘都支援
- 官方建議不要使用或為客戶提供已經EOS的版本功能

---

## 功能

[PostgreSQL: Feature Matrix](https://www.postgresql.org/about/featurematrix/)

---

## 語系

- 不支援Big 5編碼
<style scoped> table {
width: 90%; 
font-size: 14px;
} </style>

| Name           | Description                       | Language                       | Server? | Aliases                |
| -------------- | --------------------------------- | ------------------------------ | ------- | ---------------------- |
| `BIG5`         | Big Five                          | Traditional Chinese            | No      | `WIN950`, `Windows950` |
| `EUC_CN`       | Extended UNIX Code-CN             | Simplified Chinese             | Yes     |                        |
| `EUC_JP`       | Extended UNIX Code-JP             | Japanese                       | Yes     |                        |
| `EUC_JIS_2004` | Extended UNIX Code-JP, JIS X 0213 | Japanese                       | Yes     |                        |
| `EUC_KR`       | Extended UNIX Code-KR             | Korean                         | Yes     |                        |
| `EUC_TW`       | Extended UNIX Code-TW             | Traditional Chinese, Taiwanese | Yes     |                        |
| `GB18030`      | National Standard                 | Chinese                        | No      |                        |
| `GBK`          | Extended National Standard        | Simplified Chinese             | No      |                        |

---
<style scoped>ul { 
font-size:20px;
}
</style>
## Postgres維護工具
**Community tools：**
 - pg_：內建工具[3,5]
 - psql：執行 SQL 查詢和管理資料庫[3,4]
 - pgAdmin：GUI工具[3,6]
 - pgPool-ll：讀寫分離、負載平衡、故障轉移[7]
 - pgBackRest：備份還原[7]
 
 **Enterprise tools💰：**
 - PostgreSQL Enterprise Manager(PEM)：企業版pgAdmin[3]
 - Backup and Recovery Manager(Barman)：備份還原[7]
 - Failover Manager：主備架構的自動故障轉移[7]
 
**其他：**
 - Prometheus+Gafana：效能監控與儀表板[8]

---
## Reference

1. [版本說明](https://www.postgresql.org/support/versioning/)
2. [安全性修補](https://www.postgresql.org/support/security/)
3. [版本表格說明文件](https://www.postgresql.org/message-id/flat/20200907092247.GA3967@c720-r342378)
4. [版本功能表](https://www.postgresql.org/about/featurematrix/)
5. [支援語系](https://docs.postgresql.tw/server-administration/localization/character-set-support)
6. [PostgreSQL: 軟體目錄](https://www.postgresql.org/download/product-categories/)
7. [PostgreSQL管理工具](PostgreSQL管理工具.md)
8. [PostgreSQL連線設定](PostgreSQL連線設定.md)
9. [管理資料庫](管理資料庫.md)
10. [Postgres使用者管理](Postgres使用者管理.md)
11. [Postgres高可用性配置](Postgres高可用性配置.md)
12. [Postgres監控資料庫](Postgres監控資料庫.md)
