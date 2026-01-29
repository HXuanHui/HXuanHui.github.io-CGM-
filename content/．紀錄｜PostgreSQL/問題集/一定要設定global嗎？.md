---
title: 一定要設定global嗎？
draft: false
maturity: stone
---

簡單來說：**技術上不是「絕對」必要，但實務上「強烈建議」要有。**

在 `pgbackrest.conf` 中，`[global]` 區塊的作用是定義**所有 Stanza 共用的預設參數**。如果沒有 `[global]`，你的設定檔會變得非常冗長且難以維護。

以下是針對這個問題的詳細拆解：

---

### 1. 為什麼大家都會寫 `[global]`？

`pgBackRest` 的設計遵循「繼承機制」。如果你有 3 個 Stanza，而你沒有設定 `[global]`，你的設定檔會變成這樣：

| **有 [global] (推薦)**                                                                                                   | **沒有 [global] (冗長)**                                                                                                                                      |
| --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ini [global] repo1-path=/var/lib/pgbackrest process-max=4 [db_sales] pg1-path=/data/sales [db_hr] pg1-path=/data/hr` | `ini [db_sales] repo1-path=/var/lib/pgbackrest process-max=4 pg1-path=/data/sales [db_hr] repo1-path=/var/lib/pgbackrest process-max=4 pg1-path=/data/hr` |
|                                                                                                                       |                                                                                                                                                           |

**缺點：** 當你想把備份路徑從 `/var/lib/pgbackrest` 改到 `/mnt/backup` 時，沒有 `[global]` 的話，你必須手動修改每一個 Stanza。這極易出錯。

---

### 2. 如果「只有一個 Stanza」，可以不寫 global 嗎？

**可以。** 你可以把所有參數都塞進 `[cg_test_prodcut]` 裡面：

Ini, TOML

```
[cg_test_prodcut]
pg1-path=/var/lib/postgresql/16/main
repo1-path=/var/lib/pgbackrest
repo1-retention-full=2
log-level-console=info
process-max=2
```

這樣跑起來完全沒問題。

---

### 3. 什麼參數「必須」放在 global？

雖然大部分參數可以放在 Stanza 裡，但有些參數邏輯上通常只存在於 `[global]`：

- **多倉庫設定 (`repo2-path` 等)：** 如果你同時備份到本地和 S3。
    
- **鎖定路徑 (`lock-path`)：** 防止多個進程衝突。
    
- **共用效能參數 (`process-max`)：** 決定這台機器可以調用多少 CPU 核心。
    

---

### 4. 官方的潛規則

雖然官方手冊沒說「沒寫 global 就會報錯」，但 pgBackRest 的**所有官方範例**都是從 `[global]` 開始的。這已經成為 PostgreSQL 社群的一種標準格式。

> 總結建議：
> 
> 即使你現在只有一個資料庫，我也建議保留 [global]。這能讓你一眼看出：「哪些是關於這台伺服器的環境設定（Global）」，「哪些是關於這個資料庫的路徑設定（Stanza）」。


#postgreSQL 