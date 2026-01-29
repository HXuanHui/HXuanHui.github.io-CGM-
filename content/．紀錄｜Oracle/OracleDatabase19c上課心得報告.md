---
marp: true
theme: gaia
paginate: true
style: |-
  section {
    font-size: 28px;
    padding: 90px;
  }
  h1, h2 {
    color: #f05b62; /* Oracle red */
  }
  ul, ol {
    list-style-position: inside;
  }
  img{display: block; margin:0 auto;max-width: 100%;max-height: 80%;object-fit: contain;
  }
  table{font-size:24px}
draft: false
maturity: Tree
---
![bg left:40% opacity:0.5](．紀錄｜Oracle/Oracle封面.png) 

# Oracle Database 19c 上課心得報告

**黃暄惠**  
  

---

## 課程的重點學習內容

課程涵蓋Oracle 19c的多租戶容器資料庫（CDB）和可插拔資料庫（PDB），我挑選三個在實作和後續任務中印象最深的內容分享。
 - **資料字典建立**
 - **使用者與角色管理**
 - **交易與恢復機制**


---

### 資料字典建立


兩種建立方式：  

- 使用DBCA（Database Configuration Assistant）：自動執行catalog.sql和catproc.sql腳本，一次建立好資料字典視圖、內建套件和必要功能。
- 手動使用CREATE DATABASE SQL命令：需自行執行catalog.sql和catproc.sql來補齊字典，否則資料庫功能不完整。  


```sql
-- 範例：手動建立後執行腳本
@$ORACLE_HOME/rdbms/admin/catalog.sql -- catalog建立核心資料字典
@$ORACLE_HOME/rdbms/admin/catproc.sql -- catproc建立PL/SQL內建套件
```

---

### 使用者與角色管理


- **使用者（User）**：是資料庫中可登入的帳號實體。每個使用者帳號通常擁有一個預設的 Schema，作為使用者建立和擁有的資料庫物件的集合。
- **角色（Role）**：是多個相關權限的集合，簡化權限管理。 必須透過GRANT命令分配給使用者才生效，不能直接登入。  
- 權限分為**系統權限**（如CREATE SESSION，允許登入）和**物件權限**（如SELECT ON特定表格）。  

---

### 使用者與角色管理


``` sql
CREATE USER myuser --可登入的實體
GRANT DBA TO myuser --Role授權給User
GRANT SELECT ON table TO anotheruser --將物件權限直接授權給其他User
``` 

---

### 交易與恢復機制

|方面|Rollback (回滾)|Rollforward (前滾)|
|---|---|---|
|**記錄什麼**|如何取消變更 |如何重做變更|
|**儲存位置**|Undo segments (undo表空間)|Redo log files|
|**主要用途**|交易回滾、讀取一致性、閃回、失敗恢復|資料庫前滾（恢復已提交變更）、instance/media recovery|
|**SQL命令**|`ROLLBACK;`|無直接命令（自動或在`RECOVER`中使用）|

---

### 交易與恢復機制(Cont'd)

| 方面        | Instance Recovery     | Media Recovery             |
| --------- | --------------------- | -------------------------- |
| **觸發原因**  | 實例崩潰（e.g., 電源中斷、軟體故障） | 實體媒體損壞（e.g., 資料檔遺失、磁碟故障）   |
| **執行方式**  | 自動（SMON處理）            | 手動（DBA使用RMAN或SQL）          |
| **依賴資源**  | 現有redo log和undo data  | 備份（cold/hot）+archive log   |
| **恢復範圍**  | 崩潰前的最新點（無資料遺失）        | 備份點到最新log點（可能有遺失）          |
| **時間複雜度** | 通常較快（自動、內存操作）         | 較慢（需還原備份並應用log）            |
| **命令/工具** | 無需手動命令（自動）            | `RECOVER DATABASE;` (RMAN) |


---

## 課程內容與 PostgreSQL 建置的關聯

在建置PostgreSQL資料庫的任務中，我發現許多概念與Oracle相通，這強化了我的記憶；但也有些差異，讓我反思課程內容。  

- **相似部分**：

| **項目** | **Oracle** | **PostgreSQL** |
|------|-------------------|-----------------|
| **Tablespace** | 用來管理儲存空間，分為永久和臨時Tablespace。 | 同樣用來管理儲存空間，類似永久和臨時Tablespace。 |
| **控制檔案** | Control File儲存資料庫元資料，如檔案位置和狀態。 | pg_control檔案類似，儲存元資料（如檢查點資訊），但格式和功能與Oracle不同，較簡化。



---
- **差異部分**：


| **項目**            | **Oracle**                       | **PostgreSQL**            |
|---------------------|---------------------------------------------|------------------------------------------|
| **建置工具**        | 使用DBCA或SQL命令 | 使用initdb命令初始化，搭配postgresql.conf設定檔，流程更簡潔。 |
| **角色管理**        | 角色（Role）為權限集合，需透過GRANT分配，支持系統權限和物件權限。 | ROLE可嵌套分配權限，無明確區分系統/物件權限。 |
| **複雜度與靈活性**  | Oracle，適合企業環境，預設值需調校，但功能全面。 | PostgreSQL更簡潔，預設值較適合小型環境，但需手動配置權限與參數。 |
