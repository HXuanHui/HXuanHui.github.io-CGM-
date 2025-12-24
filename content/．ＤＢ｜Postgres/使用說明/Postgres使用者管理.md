---
theme:
  - default
class: []
paginate: true
header: Postgres使用者管理
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


# Postgres使用者管理
黃暄惠 2024/11

---

## 創建Role
![](．紀錄｜PostgreSQL/picture/createrole.png)

---
### 表示權限的Role

```sql
CREATE ROLE test WITH
	NOLOGIN NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT NOREPLICATION 
	NOBYPASSRLS CONNECTION LIMIT -1;
```

![width:200px height:320px](．紀錄｜PostgreSQL/picture/role.png)

---
### User

```sql
CREATE ROLE test_user WITH
	LOGIN
	NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT NOREPLICATION
	NOBYPASSRLS CONNECTION LIMIT -1
	PASSWORD 'xxxxxx';
```

⭐CREATE USER是另一種語法。在Postgres裡面如果沒有特別指定，Role被創建之後僅是代表一組權限(WITH NOLOGIN)，如果要用做登入的話就是向上面例子一樣加入WITH lOGIN，輸入CREATE USER也可以達成一樣的效果。


---
### Role 群組

```sql
---權限賦予
GRANT test TO test_user;

---指定新角色創建時就屬於另一角色，並繼承其權限。
CREATE ROLE test_user WITH
	LOGIN
	NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT NOREPLICATION
	NOBYPASSRLS CONNECTION LIMIT -1
	PASSWORD 'xxxxxx'
	IN ROLE test;
```


---
## 修改現有的Role

```sql
ALTER ROLE test_user
	RENAME TO test_superuser;

ALTER ROLE test_superuser
	SUPERUSER
	CREATEDB 
	CREATEROLE 
	INHERIT 
	REPLICATION 
	BYPASSRLS;
```

![bg right 80%](．紀錄｜PostgreSQL/picture/alterRole.png)

---
## 刪除現有的Role

例子:

```sql
DROP role test_superuser;
```

當有權限或是物件的所有權在role身上的時候，移除會出現錯誤訊息，要把上面的權限進行移除與/或轉讓到其他的role之後才能刪除。

---
## 查看資料庫上面現有的Role

在PSQL命令列中，可以輸入'\du'指令

![](．紀錄｜PostgreSQL/picture/du.png)


---
## Role的權限繼承

在使用者執行操作的時候，PostgreSQL會檢查使用者的Role是否有權執行該指令，如果沒有明確被授權的話，PostgreSQL會去尋找這個Role所屬的Role以及PUBLIC role。

如果這樣依然沒有找到對應的權限，則會繼續往上一層有設定INHERIT的role去找，最後沒有找到權限的話，操作就會被拒絕。

在定義Role的時候可以加入INHERIT/NOINHERIT來決定role所帶的權限是否能繼承給底下的Role。

