# 01 - SQL 基础（SQL Fundamentals）

> 📺 课程时间：00:04 – 1:06:22
> ✅ 学习状态：进行中

---

## 1.1 数据库简介（00:04）

### 为什么需要数据库？

**电子表格（Excel/Google Sheets）的局限：**
- 数据量大时变慢（几万行就开始卡）
- 多人同时编辑容易冲突
- 数据冗余严重（同一首歌出现在多个播放列表里就要复制多行）
- 缺乏数据完整性保障（你删了一个歌手的行，他的歌怎么办？）
- 查询能力有限（想找"2023年发行的摇滚专辑中评分最高的10张"很麻烦）

**数据库管理系统（DBMS）的优势：**
- 能处理百万甚至数十亿条数据
- 支持并发访问（成百上千人同时操作）
- 通过表的设计消除冗余
- 保证数据完整性和一致性（约束、事务）
- 强大的查询语言（SQL）灵活提取任意数据

### 关系型数据库（Relational Database）

核心思想：**数据以"表（Table）"的形式组织，表之间通过关系关联。**

```
想象中的"一张大表"（电子表格思维）：
┌────────┬──────────┬───────────┬───────────┬────────┐
│ Album  │ Artist   │ Genre     │ Track     │ Rating │
├────────┼──────────┼───────────┼───────────┼────────┤
│ Thriller│ MJ      │ Pop       │ Beat It   │ 5      │
│ Thriller│ MJ      │ Pop       │ Billie Jean│ 5     │  ← 大量重复！
└────────┴──────────┴───────────┴───────────┴────────┘

数据库思维（多表关联）：
 albums 表        artists 表       tracks 表
 ┌────┬──────┐   ┌────┬─────┐   ┌────┬────────────┬──────────┐
 │ id │name  │   │ id │name │   │ id │ name       │ album_id │
 ├────┼──────┤   ├────┼─────┤   ├────┼────────────┼──────────┤
 │ 1  │Thrill│   │ 1  │ MJ  │   │ 1  │ Beat It    │ 1        │
 └────┴──────┘   └────┴─────┘   │ 2  │ Billie Jean│ 1        │
                                 └────┴────────────┴──────────┘
```

### SQL 是什么？

**SQL = Structured Query Language（结构化查询语言）**

- 操作关系型数据库的统一语言
- 分为四大类：
  - **DQL**（Data Query）：`SELECT` — 查询数据
  - **DML**（Data Manipulation）：`INSERT`, `UPDATE`, `DELETE` — 操作数据
  - **DDL**（Data Definition）：`CREATE`, `ALTER`, `DROP` — 定义结构
  - **DCL**（Data Control）：`GRANT`, `REVOKE` — 控制权限

---

## 1.2 环境配置（13:36）

### SQLite 简介

SQLite 是世界上最广泛部署的数据库引擎：
- **轻量级**：不需要服务器，直接读写文件
- **零配置**：不需要安装、配置、管理
- **自包含**：整个数据库就是一个 `.db` 文件
- **适用场景**：移动 App、桌面软件、原型开发、学习

> 💡 课程用 SQLite 入门，学完后再接触 MySQL/PostgreSQL 会更轻松。

### 安装与配置

**macOS（你的环境）：**
```bash
# SQLite 通常已预装，检查
sqlite3 --version

# VS Code 安装插件
# 搜索安装：SQLite Viewer 或 SQLite
```

**创建第一个数据库：**
```bash
# 创建并进入数据库（文件不存在会自动创建）
sqlite3 mydatabase.db

# 在 sqlite3 命令行中：
.tables          # 查看所有表
.schema          # 查看表结构
.mode column     # 列对齐显示
.headers on      # 显示列名
.quit            # 退出
```

### 课程数据集

CS50 课程使用一个 `longlist.db` 数据库，包含纽约时报畅销书榜单数据。表中包含书的标题、作者、年份、评分、页数、出版社等信息。

---

## 1.3 基本查询（21:51）

### SELECT 语句

```sql
-- 查询所有列
SELECT * FROM longlist;

-- 查询指定列
SELECT title, author FROM longlist;

-- 查询时可以看到表中有哪些数据
```

### WHERE 条件过滤

```sql
-- 精准过滤
SELECT title, author
FROM longlist
WHERE year = 2023;

-- 多个条件
SELECT title, author
FROM longlist
WHERE year = 2023 AND genre = 'Fiction';
```

**WHERE 子句执行逻辑：**
1. `FROM`：定位到表
2. `WHERE`：逐行检查条件，只保留满足的行
3. `SELECT`：选出指定的列

### LIMIT 限制结果

```sql
-- 只看前 10 条
SELECT title FROM longlist LIMIT 10;

-- 先过滤再限制
SELECT title, rating
FROM longlist
WHERE year = 2023
LIMIT 5;
```

### SQL 书写规范

- 关键字习惯用**大写**（`SELECT`, `FROM`, `WHERE`），但不是必须的
- 语句以 `;` 结尾
- 字符串用**单引号** `' '`
- 换行和缩进自由，推荐每个子句一行，提高可读性

---

## 1.4 过滤与运算符（32:18）

### 比较运算符

| 运算符 | 含义 | 示例 |
|--------|------|------|
| `=` | 等于 | `WHERE year = 2023` |
| `!=` 或 `<>` | 不等于 | `WHERE genre != 'Fiction'` |
| `>` | 大于 | `WHERE rating > 4.0` |
| `<` | 小于 | `WHERE pages < 300` |
| `>=` | 大于等于 | `WHERE year >= 2020` |
| `<=` | 小于等于 | `WHERE rating <= 3.5` |

```sql
-- 实战示例
SELECT title, rating, pages
FROM longlist
WHERE rating > 4.0 AND pages < 300;
-- 找出「评分高且篇幅短」的书
```

### 逻辑运算符

| 运算符 | 含义 | 说明 |
|--------|------|------|
| `AND` | 与 | 所有条件都要满足 |
| `OR` | 或 | 满足任一条件即可 |
| `NOT` | 非 | 取反 |

```sql
-- AND：条件更严格，结果更少
SELECT title FROM longlist
WHERE year >= 2020 AND year <= 2023 AND genre = 'Fiction';

-- OR：条件更宽松，结果更多
SELECT title FROM longlist
WHERE genre = 'Fiction' OR genre = 'Nonfiction';

-- NOT：排除
SELECT title FROM longlist
WHERE NOT genre = 'Nonfiction';
-- 等价于：WHERE genre != 'Nonfiction'

-- 混合使用：注意优先级！
-- AND 优先级高于 OR，所以要用括号明确
SELECT title FROM longlist
WHERE (genre = 'Fiction' OR genre = 'Mystery')
  AND rating >= 4.0;
-- 找「Fiction 或 Mystery 中评分 >= 4.0 的」
-- 不加括号的话，AND 会在 OR 之前执行，逻辑就变了
```

### LIKE 模式匹配（通配符）

```sql
-- % 匹配任意数量字符（包括 0 个）
SELECT title FROM longlist
WHERE title LIKE 'The%';
-- 匹配：The Great Gatsby, The Alchemist, The...

SELECT title FROM longlist
WHERE title LIKE '%Love%';
-- 匹配：The Love Song, To Love and Be Loved, Love Story...

-- _ 匹配恰好 1 个字符
SELECT title FROM longlist
WHERE title LIKE '____';
-- 匹配恰好 4 个字符的书名

SELECT title FROM longlist
WHERE title LIKE 'T_e%';
-- 匹配：The..., Tie..., Toe...
```

### NULL 值处理

`NULL` 表示「未知」或「缺失」，**不是 0，不是空字符串**。

```sql
-- 错误写法！NULL 不能用 = 比较
SELECT * FROM longlist WHERE translator = NULL;  -- ❌ 永远返回空

-- 正确写法
SELECT * FROM longlist WHERE translator IS NULL;      -- ✅ 没有译者的
SELECT * FROM longlist WHERE translator IS NOT NULL;  -- ✅ 有译者的
```

> ⚠️ **NULL 的核心规则**：任何与 NULL 的比较结果都是 NULL（未知），不是 TRUE 也不是 FALSE。
> `NULL = NULL` → `NULL`（未知），不是 `TRUE`！

---

## 1.5 范围与模式匹配（46:59）

### BETWEEN ... AND ...

```sql
-- 范围查询（包含边界）
SELECT title, year FROM longlist
WHERE year BETWEEN 2020 AND 2023;
-- 等价于：WHERE year >= 2020 AND year <= 2023

-- 也可以用于数值
SELECT title, pages FROM longlist
WHERE pages BETWEEN 200 AND 400;

-- 用于文本（字典序）
SELECT title FROM longlist
WHERE title BETWEEN 'A' AND 'C';
-- 'A' 开头到 'B' 开头的（不包含 'C' 开头）
```

### IN (...) 列表匹配

```sql
-- 匹配列表中的任意值
SELECT title, genre FROM longlist
WHERE genre IN ('Fiction', 'Mystery', 'Science Fiction');

-- 等价于：
-- WHERE genre = 'Fiction' OR genre = 'Mystery' OR genre = 'Science Fiction'
-- 显然 IN 更简洁！

-- NOT IN：排除列表
SELECT title, genre FROM longlist
WHERE genre NOT IN ('Nonfiction', 'Biography');
```

### 查询优先级

```
1. 括号 ()
2. 比较运算符 = != > < >= <=
3. NOT
4. AND
5. OR
```

> 💡 **记住：有疑惑就加括号！** 既让自己意图清晰，也让数据库不误解析。

---

## 1.6 排序（57:53）

### ORDER BY 基础

```sql
-- 按评分升序（默认 ASC）
SELECT title, rating FROM longlist
ORDER BY rating;

-- 按评分降序
SELECT title, rating FROM longlist
ORDER BY rating DESC;

-- 按年份降序，最新的排前面
SELECT title, year FROM longlist
ORDER BY year DESC;
```

### 多列排序

```sql
-- 先按评分降序，评分相同的再按页数升序
SELECT title, rating, pages FROM longlist
ORDER BY rating DESC, pages ASC;

-- 先按年份降序，再按书名升序
SELECT title, year FROM longlist
ORDER BY year DESC, title ASC;
```

### ORDER BY 与 WHERE 组合

```sql
-- 执行顺序：FROM → WHERE → SELECT → ORDER BY → LIMIT
SELECT title, rating, year
FROM longlist
WHERE year >= 2020 AND rating IS NOT NULL
ORDER BY rating DESC
LIMIT 10;
-- 2020 年以来评分最高的 10 本书
```

> 🔑 **执行顺序理解**（逻辑顺序，非实际执行）：
> `FROM`（找表）→ `WHERE`（过滤行）→ `SELECT`（选列）→ `ORDER BY`（排序）→ `LIMIT`（截取）

---

## 1.7 聚合函数（1:06:22）

### 五大聚合函数

```sql
-- COUNT：计数
SELECT COUNT(*) FROM longlist;
-- 总共有多少本书

SELECT COUNT(translator) FROM longlist;
-- 有译者信息的书有多少（NULL 不计入）

-- AVG：平均值
SELECT AVG(rating) FROM longlist;
-- 平均评分

-- ROUND 配合使用，保留小数
SELECT ROUND(AVG(rating), 2) FROM longlist;

-- MAX / MIN：最大/最小值
SELECT MAX(pages), MIN(pages) FROM longlist;
-- 最长的和最短的书的页数

-- SUM：求和
SELECT SUM(votes) FROM longlist;
-- 所有书的总投票数
```

### 聚合函数 + WHERE

```sql
-- 先过滤，再聚合
SELECT COUNT(*), AVG(rating)
FROM longlist
WHERE year = 2023;
-- 2023 年出版的书有多少本，平均评分多少

-- 聚合结果无法在 WHERE 中使用
-- WHERE AVG(rating) > 4.0;  ❌ 错误！
-- WHERE 在聚合之前执行，此时还没有 AVG 的结果
-- 想过滤聚合结果？→ 用 HAVING（后面课程会讲）
```

### 聚合函数的行为要点

| 函数 | 忽略 NULL？ | 适用于 | 备注 |
|------|------------|--------|------|
| `COUNT(*)` | 不忽略 | 任意类型 | 统计行数 |
| `COUNT(col)` | 忽略 | 任意类型 | 统计某列非 NULL 数 |
| `AVG(col)` | 忽略 | 数值 | NULL 不计入分母 |
| `SUM(col)` | 忽略 | 数值 | |
| `MAX(col)` | 忽略 | 数值/文本/日期 | 文本按字典序 |
| `MIN(col)` | 忽略 | 数值/文本/日期 | |

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记、心得、遇到的坑 -->



---

## 📝 本章速查

```sql
-- ==================== 基本查询 ====================
SELECT col1, col2 FROM table_name;
SELECT * FROM table_name;
SELECT DISTINCT col FROM table_name;        -- 去重

-- ==================== 条件过滤 ====================
SELECT * FROM t WHERE col = 'value';
SELECT * FROM t WHERE col != 'value';
SELECT * FROM t WHERE col > 10 AND col < 50;
SELECT * FROM t WHERE col1 = 'A' OR col1 = 'B';
SELECT * FROM t WHERE NOT col = 'X';
SELECT * FROM t WHERE col LIKE 'prefix%';     -- % 任意字符
SELECT * FROM t WHERE col LIKE '_attern';     -- _ 一个字符
SELECT * FROM t WHERE col IS NULL;
SELECT * FROM t WHERE col IS NOT NULL;
SELECT * FROM t WHERE col BETWEEN 10 AND 50;
SELECT * FROM t WHERE col IN ('A', 'B', 'C');

-- ==================== 排序 ====================
SELECT * FROM t ORDER BY col1 ASC;            -- 升序（默认）
SELECT * FROM t ORDER BY col1 DESC;           -- 降序
SELECT * FROM t ORDER BY col1 DESC, col2 ASC; -- 多列排序

-- ==================== 聚合 ====================
SELECT COUNT(*), COUNT(col) FROM t;
SELECT AVG(col), ROUND(AVG(col), 2) FROM t;
SELECT MAX(col), MIN(col), SUM(col) FROM t;

-- ==================== 组合使用 ====================
SELECT col1, col2, AVG(col3)
FROM table_name
WHERE condition
ORDER BY col1 DESC
LIMIT 10;
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
| 2 | | |
