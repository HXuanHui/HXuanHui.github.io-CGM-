#postgreSQL 
在 psql 中切換資料庫，有幾種方法：

## 主要方法

**1. 使用 `\c` 或 `\connect` 命令**

```sql
\c database_name
-- 或
\connect database_name
```

**2. 指定使用者名稱切換**

```sql
\c database_name username
-- 或
\connect database_name username
```

**3. 完整連線語法**

```sql
\c database_name username host port
```

## 實際操作範例

```sql
-- 查看目前連線的資料庫
\conninfo

-- 列出所有可用的資料庫
\l

-- 切換到 postgres 資料庫
\c postgres

-- 切換到 mydb 資料庫，使用 myuser 使用者
\c mydb myuser

-- 切換後會顯示類似訊息：
-- You are now connected to database "mydb" as user "myuser".
```

## 補充說明

**重要觀念**：

- PostgreSQL 中每個連線只能連接到一個資料庫，不像某些資料庫系統可以跨資料庫查詢
- 切換資料庫實際上是建立新的連線並關閉舊連線
- 如果切換失敗，原來的連線會保持不變

**實用技巧**：

- 使用 `\l` 先查看有哪些資料庫可以切換
- 使用 `\conninfo` 確認目前連線狀態
- 如果需要經常在多個資料庫間切換，可以開啟多個 psql 視窗

這個功能對於管理多個資料庫或在開發環境中測試不同專案時特別有用。你在學習過程中會經常用到這個命令。


# 複習與問題
1.[user可以連縣adb，也可以連線bdb，但是連入adb之後不可以＼c切換至bdb，是不是辦不到？](user可以連縣adb，也可以連線bdb，但是連入adb之後不可以＼c切換至bdb，是不是辦不到？.md)