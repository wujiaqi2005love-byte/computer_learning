# 06 - 性能与事务（Performance & Transactions）

> 📺 课程时间：7:31:09 – 9:01:45
> ✅ 学习状态：进行中

---

## 6.1 索引基础（Index）

### 什么是索引？

**索引（Index）= 数据库表中一列或多列的"目录"。** 就像书的目录让你直接翻到某一章，索引让数据库**直接跳到目标行**，而不是逐行扫描全表。

```
没有索引（全表扫描）：            有索引（B-Tree 查找）：
SELECT * FROM books                SELECT * FROM books
WHERE title = 'The Shining';       WHERE title = 'The Shining';

遍历 100 万行                        B-Tree 查找 → 3~4 步定位
→ 找到匹配行                        → 直接跳到目标行
→ 耗时：可能几秒                    → 耗时：毫秒级
```

### 为什么索引快？——B-Tree 数据结构

索引底层使用 **B-Tree（平衡多路搜索树）**：

```
                    [Kafka, Orwell]
                   /        |        \
          [Austen, Bradbury] [King, Lee] [Tolkien, Wells]
            /    \    \        /   \      /    \    \
          ...   ...  ...    ...   ...   ...   ...  ...

查找 'Tolkien'：根节点比较 → 走右分支 → 叶子节点 → 找到行指针
时间复杂度：O(log n)，100 万行只需约 20 次比较
```

### 创建索引

```sql
-- 单列索引（最常用）
CREATE INDEX idx_books_title ON books(title);

-- 复合索引（多列组合查询）
CREATE INDEX idx_books_author_year ON books(author_id, year);

-- 唯一索引（同时保证唯一性）
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- 部分索引（只索引满足条件的行，省空间）
CREATE INDEX idx_active_orders ON orders(customer_id)
    WHERE status = 'active';

-- 删除索引
DROP INDEX idx_books_title;
```

### 什么时候建索引？

| ✅ 应该建索引 | ❌ 不应该建索引 |
|-------------|---------------|
| WHERE 子句频繁使用的列 | 频繁更新的列（每次 UPDATE 也要更新索引） |
| JOIN 的 ON 条件列（通常是 FK） | 选择性低的列（如性别只有 M/F 两种） |
| ORDER BY 排序的列 | 小表（全表扫描更快） |
| 复合查询的多列组合 | 几乎不在 WHERE 中出现的列 |

### 索引的代价

```
✅ 加速 SELECT 查询（读）
❌ 拖慢 INSERT / UPDATE / DELETE（写，因为要同时更新索引）
❌ 占用额外的磁盘空间
💡 原则：为读多写少的场景建索引，不是越多越好
```

---

## 6.2 EXPLAIN QUERY PLAN

### 查看查询计划

`EXPLAIN QUERY PLAN` 告诉你数据库**如何执行**一条查询——是否用了索引、扫描了多少行。

```sql
EXPLAIN QUERY PLAN
SELECT * FROM books WHERE title = 'The Shining';

-- 输出解读：
-- SCAN TABLE books           → 全表扫描（慢）
-- SEARCH TABLE books USING INDEX idx_books_title (title=?)  → 索引查找（快）
```

关键术语：
- `SCAN TABLE` → 全表扫描（没有用索引）
- `SEARCH TABLE ... USING INDEX` → 索引查找
- `USING COVERING INDEX` → 覆盖索引（连表都不用回，最快）

### 实用技巧

```sql
-- 比较建索引前后的查询计划
EXPLAIN QUERY PLAN SELECT ... WHERE ...;          -- 建索引前
CREATE INDEX idx_xxx ON t(col);
EXPLAIN QUERY PLAN SELECT ... WHERE ...;          -- 建索引后，看变化
```

---

## 6.3 部分索引与复合索引

### 部分索引

只索引表中**满足 WHERE 条件**的行——省空间、写入更快。

```sql
-- 只索引活跃订单（假设 90% 的订单已完成，查活跃订单最频繁）
CREATE INDEX idx_active_orders_customer
ON orders(customer_id) WHERE status = 'active';
```

### 复合索引

多列组合成一个索引。**列的顺序很重要！**

```sql
-- 复合索引对以下查询有效：
CREATE INDEX idx_books_author_year ON books(author_id, year);

-- ✅ 能用索引：author_id = ?（前缀列）
-- ✅ 能用索引：author_id = ? AND year = ?（全匹配）
-- ❌ 不能用索引：year = ?（跳过了前缀列 author_id）
```

> 💡 **复合索引的最左前缀原则**：复合索引 (A, B, C) 对 `WHERE A` 和 `WHERE A AND B` 有效，但对单独的 `WHERE B` 或 `WHERE C` 无效。

---

## 6.4 VACUUM

### SQLite 的存储特性

SQLite 删除数据后**不会自动释放磁盘空间**——它把空间标记为"可重用"，但文件大小不变。

```sql
-- 删除大量数据后，文件大小不变
DELETE FROM logs WHERE created_at < '2020-01-01';

-- VACUUM 重建数据库文件，回收空间
VACUUM;
```

### 什么时候用 VACUUM

| 场景 | 说明 |
|------|------|
| 大量 DELETE 之后 | 回收已释放的空间 |
| 大量 UPDATE 之后 | 整理碎片 |
| 定期维护 | 保持数据库文件紧凑 |
| 归档前 | 压缩数据库再存档 |

> ⚠️ VACUUM 会**锁住整个数据库**，期间不能读写。在生产环境中要选低峰期执行。

---

## 6.5 事务（Transactions）

### 什么是事务？

**事务 = 一组 SQL 操作的集合，这些操作要么全部成功，要么全部回滚。** 事务是保证数据一致性的核心机制。

```sql
BEGIN;                          -- 开始事务
UPDATE accounts SET balance = balance - 100 WHERE id = 1;   -- A 账户扣 100
UPDATE accounts SET balance = balance + 100 WHERE id = 2;   -- B 账户加 100
COMMIT;                         -- 提交事务（持久化）
-- 如果中间出错：ROLLBACK;       -- 回滚事务（撤销所有操作）
```

### 为什么需要事务？

银行转账场景——没有事务的灾难：

```
转账：A → B 转 100 元

Step 1: UPDATE accounts SET balance = balance - 100 WHERE id = 1;  ✅ A 扣了 100
                                    ↓ 系统崩溃！💥
Step 2: UPDATE accounts SET balance = balance + 100 WHERE id = 2;  ❌ B 没收到

结果：A 的钱消失了，B 也没收到。100 元凭空蒸发！
```

有事务：

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- 扣钱
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- 加钱
-- 如果一切正常：COMMIT;  → 两笔操作同时生效
-- 如果出错了：ROLLBACK;  → 两笔操作同时撤销，A 的钱还在
```

### 事务的三个命令

| 命令 | 作用 |
|------|------|
| `BEGIN` | 开始事务 |
| `COMMIT` | 提交——所有操作永久生效 |
| `ROLLBACK` | 回滚——撤销事务中所有操作 |

---

## 6.6 ACID 特性

事务保证数据库在任何情况下都保持一致。核心是 **ACID** 四大特性：

```
A — Atomicity（原子性）
    事务是不可分割的最小单元。要么全做，要么全不做。
    转账中扣钱和加钱必须同时成功，不能扣了钱但没加钱。

C — Consistency（一致性）
    事务执行前后，数据库都处于一致状态（满足所有约束）。
    转账前后，所有账户的总余额不变。
    A(1000) + B(500) = 1500 → A(900) + B(600) = 1500 ✅

I — Isolation（隔离性）
    并发事务之间互不干扰。
    你转你的账，我买我的东西，互不影响。
    不能出现：一个事务读到另一个事务未提交的中间状态。

D — Durability（持久性）
    一旦 COMMIT，数据就不会丢失（即使断电/崩溃）。
    提交后的数据写入磁盘，重启后仍在。
```

### 隔离性问题

| 问题 | 说明 | 示例 |
|------|------|------|
| **脏读** (Dirty Read) | 读到了另一个事务未提交的修改 | 读到 A 扣了钱但事务还没提交（可能回滚） |
| **不可重复读** (Non-repeatable Read) | 同一事务中两次读取结果不同 | SELECT → 别的事务 UPDATE + COMMIT → 再 SELECT，值变了 |
| **幻读** (Phantom Read) | 同一事务中两次范围查询结果不同 | SELECT COUNT(*) → 别的事务 INSERT + COMMIT → 再 SELECT，多了行 |

### 隔离级别

| 级别 | 脏读 | 不可重复读 | 幻读 | 性能 |
|------|------|-----------|------|------|
| READ UNCOMMITTED | ✅ 可能 | ✅ 可能 | ✅ 可能 | 最快 |
| READ COMMITTED | ❌ | ✅ 可能 | ✅ 可能 | |
| REPEATABLE READ | ❌ | ❌ | ✅ 可能 | |
| SERIALIZABLE | ❌ | ❌ | ❌ | 最慢 |

> 💡 SQLite 默认使用 **SERIALIZABLE**（最高隔离级别）。虽然绝对安全，但并发写入性能较差——同一时间只有一个连接能写。

---

## 6.7 SQLite 并发特性

### WAL 模式（Write-Ahead Logging）

SQLite 3.7+ 支持 WAL 模式，允许多个读者 + 一个写者同时操作：

```sql
-- 启用 WAL 模式（推荐）
PRAGMA journal_mode = WAL;

-- 效果：
-- 读操作不阻塞写操作
-- 写操作不阻塞读操作
-- 但同一时间仍然只有一个写者
```

### SQLite 并发限制

```
SQLite 的并发模型：
  多个连接可以同时读 ✅
  同一时间只有一个连接能写 ⚠️
  写操作会锁住整个数据库（不是只锁表或行）
```

> 💡 对于 Web 应用的高并发写入场景，SQLite 不是最佳选择。MySQL/PostgreSQL 支持行级锁和更好的并发。

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->



---

## 📝 本章速查

```sql
-- ==================== 索引 ====================
-- 创建索引
CREATE INDEX idx_name ON table_name(column);
CREATE INDEX idx_multi ON table_name(col1, col2);              -- 复合索引
CREATE UNIQUE INDEX idx_unique ON table_name(column);          -- 唯一索引
CREATE INDEX idx_partial ON table_name(column) WHERE condition;-- 部分索引

-- 查看索引
SELECT * FROM sqlite_master WHERE type = 'index';
PRAGMA index_list('table_name');

-- 分析查询
EXPLAIN QUERY PLAN SELECT ...;

-- 删除索引
DROP INDEX idx_name;

-- ==================== VACUUM ====================
VACUUM;  -- 回收空间，整理碎片

-- ==================== 事务 ====================
BEGIN;
-- SQL 操作...
COMMIT;     -- 或 ROLLBACK;

-- ==================== 设置 ====================
PRAGMA journal_mode = WAL;       -- 启用 WAL（更好的并发读）
PRAGMA foreign_keys = ON;        -- 启用外键约束
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
| 2 | | |
