# 02 - 数据库设计（Database Design）

> 📺 课程时间：1:17:08 – 1:49:13
> ✅ 学习状态：进行中

---

## 2.1 去重与数据冗余（1:17:08）

### DISTINCT — 去重查询

当查询结果出现大量重复时，用 `DISTINCT` 去重：

```sql
-- 有重复的结果
SELECT genre FROM books;
-- Fiction, Fiction, Fiction, Nonfiction, Fiction, ...

-- 去重：每种类型只出现一次
SELECT DISTINCT genre FROM books;
-- Fiction, Nonfiction, Mystery, Science Fiction, Horror

-- 多列去重：组合值同时相同才去重
SELECT DISTINCT author, genre FROM books;
-- 同作者同类型的只保留一行
```

### 数据冗余 — 为什么它是坏的

假设把作者信息和书籍信息放在同一张表里：

```
❌ 单表设计（有冗余）：
┌──────────────────┬──────────────────┬───────────────┬──────┬─────┐
│ title            │ author           │ author_birth  │ year │ ... │
├──────────────────┼──────────────────┼───────────────┼──────┼─────┤
│ The Shining      │ Stephen King     │ 1947          │ 1977 │ ... │
│ It               │ Stephen King     │ 1947          │ 1986 │ ... │  ← author_birth 重复了！
│ The Martian      │ Andy Weir        │ 1972          │ 2011 │ ... │
│ Project Hail Mary│ Andy Weir        │ 1972          │ 2021 │ ... │  ← 又重复了！
└──────────────────┴──────────────────┴───────────────┴──────┴─────┘
```

**冗余导致的四大问题：**

| 问题 | 说明 |
|------|------|
| 🔁 **数据重复** | 同一信息存多次，浪费空间 |
| 🔄 **更新异常** | 改 Stephen King 的出生年份要改所有行，漏一行数据就不一致 |
| ➕ **插入异常** | 想加一个新作者，但他还没有书，插不进去（NOT NULL 约束拦着） |
| ➖ **删除异常** | 删除某作者唯一一本书，这个作者的信息就彻底消失了 |

### 解决方式：拆表

```
✅ 拆成两张表：
     authors                        books
┌────┬──────────────┬──────┐   ┌────┬──────────────┬───────────┬──────┐
│ id │ name         │ birth│   │ id │ title        │ author_id │ year │
├────┼──────────────┼──────┤   ├────┼──────────────┼───────────┼──────┤
│ 1  │ Stephen King │ 1947 │   │ 1  │ The Shining  │ 1         │ 1977 │
│ 2  │ Andy Weir    │ 1972 │   │ 2  │ It           │ 1         │ 1986 │
└────┴──────────────┴──────┘   │ 3  │ The Martian  │ 2         │ 2011 │
                               │ 4  │ Proj Hail M. │ 2         │ 2021 │
                               └────┴──────────────┴───────────┴──────┘
                                        author_id 引用 authors.id
```

- 作者信息只存一次，没有冗余
- 改出生年份？改一行就行
- 加一个还没出书的作者？直接 INSERT 到 authors 表

---

## 2.2 数据库关系模型（1:29:24）

### 三种关系类型

```
1. 一对多（One-to-Many）       — 最常见
   作者 ──┬── 书
          │     一个作者可以写多本书，但每本书只有一个作者
          ├── 书
          └── 书

2. 多对多（Many-to-Many）      — 需要中介表
   学生 ──┬── 课程
          │     一个学生选多门课，一门课被多个学生选
          ├── 课程
          └── 课程

3. 一对一（One-to-One）        — 很少见
   用户 ──── 用户资料
              一个用户只有一个详细资料，反之亦然
```

### ER 图（Entity-Relationship Diagram）

ER 图是设计数据库的**蓝图**，用图形表示实体和它们之间的关系。

```
作者（Author）
  │
  │ 1      (一对多：一个作者 → 多本书)
  │
  ├─── 写 ───
  │
  │ N
书（Book）────────── 出版社（Publisher）
  │          N:1
  │
  │ M
  ├─── 属于 ─── 分类（Genre）
  │       N:1
```

> 💡 实际画 ER 图时，常用工具：**draw.io**、**dbdiagram.io**、甚至纸笔。课程中 Carter 老师会用图形直观展示。

### 关系建模要点

1. 找出所有**实体**（Entity）：Author, Book, Publisher, Genre → 每个实体对应一张表
2. 确定实体间的**关系**（Relationship）：写、出版、分类
3. 判断关系的**基数**（Cardinality）：1:1、1:N、N:M
4. 对于 N:M 关系：需要一个**联结表**（Junction/Join Table，后面第 5 章细讲）

---

## 2.3 主键（1:37:12）

### 什么是主键？

**主键（Primary Key）= 表中唯一标识每一行的列（或列的组合）。**

核心属性：
- ✅ **唯一性**：不能有两行拥有相同的主键值
- ✅ **非空**：主键列不能为 NULL
- ✅ **不变性**：主键值不应该随时间改变（选好了就不要改）

### 自然键 vs 代理键

| | 自然键（Natural Key） | 代理键（Surrogate Key） |
|---|---|---|
| 定义 | 数据本身就有的唯一标识 | 系统生成的 ID |
| 例子 | ISBN（书的国际标准书号）、身份证号 | 自增整数（1, 2, 3...）、UUID |
| 优点 | 有业务含义、不需要额外列 | 简单、稳定、不依赖外部 |
| 缺点 | 可能变化、可能为空、可能重复 | 无业务含义 |
| 推荐？ | ❌ 一般不推荐 | ✅ **首选** |

```sql
-- 代理键（推荐）
CREATE TABLE books (
    id INTEGER PRIMARY KEY,  -- 自增整数，纯粹用来标识行
    title TEXT,
    isbn TEXT UNIQUE          -- ISBN 设为 UNIQUE 但不做主键
);

-- 自然键
CREATE TABLE books (
    isbn TEXT PRIMARY KEY,    -- 用 ISBN 做主键
    title TEXT
);
-- 问题：万一两本书共用一个 ISBN？万一 ISBN 系统改革？
```

> 🎯 **核心原则**：主键的任务是「唯一标识一行」，不是「持有有意义的数据」。让主键简单、稳定、无意义。

### 自增主键

```sql
-- SQLite 中 INTEGER PRIMARY KEY 自动生成递增 ID
CREATE TABLE authors (
    id INTEGER PRIMARY KEY,  -- 插入时不指定 id，自动 1, 2, 3...
    name TEXT NOT NULL
);

INSERT INTO authors (name) VALUES ('Stephen King');  -- id 自动 = 1
INSERT INTO authors (name) VALUES ('Andy Weir');     -- id 自动 = 2
```

---

## 2.4 外键（1:43:25）

### 什么是外键？

**外键（Foreign Key）= 一张表里的列，指向另一张表的主键。**

外键是表与表之间的**连接点**——它告诉数据库"这一行的这个值，必须对应另一张表中真实存在的一行"。

```
books 表                    authors 表
┌────┬──────────────┬───────────┐   ┌────┬──────────────┐
│ id │ title        │ author_id │   │ id │ name         │
├────┼──────────────┼───────────┤   ├────┼──────────────┤
│ 1  │ The Shining  │ 1         │──→│ 1  │ Stephen King │
│ 2  │ It           │ 1         │──→│    │              │
│ 3  │ The Martian  │ 2         │──→│ 2  │ Andy Weir    │
└────┴──────────────┴───────────┘   └────┴──────────────┘
      author_id = 外键（FK）           id = 主键（PK）
      引用 authors 表                  被引用的表
```

### 创建外键约束

```sql
CREATE TABLE books (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    author_id INTEGER,
    FOREIGN KEY (author_id) REFERENCES authors(id)
    --  ↑ 确保 author_id 的值一定在 authors(id) 中存在
);
```

外键约束保证**参照完整性（Referential Integrity）**：
- 不能插入一个不存在的作者 ID
- 不能删除一个还有书关联的作者（取决于 `ON DELETE` 设置）

### ON DELETE 行为

```sql
FOREIGN KEY (author_id) REFERENCES authors(id)
    ON DELETE CASCADE    -- 删除作者 → 自动删除他的所有书
--  ON DELETE SET NULL   -- 删除作者 → 书的 author_id 变成 NULL
--  ON DELETE RESTRICT   -- 删除作者 → 如果还有书就阻止删除（默认）
--  ON DELETE SET DEFAULT-- 删除作者 → 设回默认值
```

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->



---

## 📝 本章速查

```sql
-- ==================== 去重 ====================
SELECT DISTINCT column FROM table;
SELECT DISTINCT col1, col2 FROM table;       -- 组合去重

-- ==================== 创建带主键的表 ====================
CREATE TABLE authors (
    id INTEGER PRIMARY KEY,                  -- 自增主键
    name TEXT NOT NULL,
    birth_year INTEGER
);

-- ==================== 创建带外键的表 ====================
CREATE TABLE books (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    author_id INTEGER,
    FOREIGN KEY (author_id) REFERENCES authors(id)
);

-- ==================== 关系类型 ====================
-- 一对多：author ──┬── books
-- 多对多：students ──┬──  enrollment  ──┬── courses
-- 一对一：user ──── user_profile

-- ==================== 主键选择原则 ====================
-- ✅ 代理键（自增整数）— 简单、稳定
-- ❌ 自然键（ISBN、身份证）— 可能变、可能空
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
| 2 | | |
