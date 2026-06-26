# 04 - Schema 设计与数据操作（Schema & Data Manipulation）

> 📺 课程时间：2:52:56 – 5:25:20
> ✅ 学习状态：进行中

---

## 4.1 分组与过滤（2:52:56）

### GROUP BY — 分组聚合

第 1 章学了聚合函数（COUNT、AVG...），但它们只能把**全部数据**汇总成一行。`GROUP BY` 让你**按分类**分别汇总。

```sql
-- 不分组：所有书的平均分
SELECT AVG(rating) FROM books;  → 一行结果

-- 分组：每种类型的平均分
SELECT genre, AVG(rating)
FROM books
GROUP BY genre;                 → 每个类型一行
```

**执行逻辑**：
1. `FROM` → 找表
2. `WHERE` → 过滤行
3. `GROUP BY` → 按指定列分组（把相同的归到一起）
4. `SELECT` → 对每组分别计算聚合
5. `ORDER BY` → 排序

### HAVING — 过滤分组

`WHERE` 过滤**行**（分组前），`HAVING` 过滤**组**（分组后）。

```sql
-- ❌ 错误：WHERE 里不能用聚合函数
SELECT genre, AVG(rating)
FROM books
WHERE AVG(rating) > 4.0   -- 报错！
GROUP BY genre;

-- ✅ 正确：HAVING 过滤分组结果
SELECT genre, AVG(rating)
FROM books
GROUP BY genre
HAVING AVG(rating) > 4.0;
```

### WHERE vs HAVING

| | WHERE | HAVING |
|---|-------|--------|
| 作用对象 | 原始**行** | 聚合后的**组** |
| 执行时机 | GROUP BY 之前 | GROUP BY 之后 |
| 能用聚合函数？ | ❌ 不能 | ✅ 能 |

```sql
-- 两者可以同时出现
SELECT genre, COUNT(*) AS cnt, AVG(rating) AS avg_r
FROM books
WHERE year >= 2000           -- 先过滤行：只要 2000 年后的书
GROUP BY genre
HAVING COUNT(*) >= 2          -- 再过滤组：只要 ≥ 2 本的类型
ORDER BY avg_r DESC;
```

---

## 4.2 Schema 设计入门（3:04:06）

### 数据库设计的步骤

```
1. 需求分析  →  用户需要什么功能？有哪些数据？
2. 实体识别  →  有哪些「东西」需要记录？（人、书、订单...）
3. 关系建模  →  这些实体之间是什么关系？（1:1 / 1:N / N:M）
4. 属性定义  →  每个实体有哪些属性？类型是什么？
5. 范式化    →  检查并消除冗余
6. 建表实现  →  CREATE TABLE + 约束
```

### 设计原则

- **一个实体一张表**：作者、书、出版社、用户、订单各一张表
- **用 FK 代替复制数据**：书的 `author_id` 引用作者，而不是存作者的名字和生日
- **约束先行**：NOT NULL、UNIQUE、FOREIGN KEY 在创建表时就想好

---

## 4.3 范式化（3:16:31）

### 三大范式

```
第一范式（1NF）：每列都是原子的（不可再分）
  ❌  tags 列存 "scifi, adventure, classic"（逗号分隔）
  ✅  拆成 book_tags 联结表，每个 tag 一行

第二范式（2NF）：非主键列完全依赖于主键（消除部分依赖）
  前提：复合主键存在时

第三范式（3NF）：非主键列不依赖于其他非主键列（消除传递依赖）
  ❌  books 表里有 author_name, author_birth
      author_birth 依赖于 author_name，不是直接依赖于 book_id
  ✅  拆出 authors 表，books 只留 author_id
```

### 范式化口诀

> "每列不拆开（1NF），每列依赖整个主键（2NF），
>  每列只依赖主键（3NF）。"

### 范式化的权衡

| | 高度范式化 | 适度反范式化 |
|---|-----------|-------------|
| 数据冗余 | 最少 | 有一些 |
| 查询复杂度 | 需要更多 JOIN | 更少 JOIN |
| 更新 | 只改一处 | 可能改多处 |
| 适用场景 | OLTP（事务处理） | OLAP（分析查询） |

---

## 4.4 建表与数据类型（3:24:46 – 3:44:14）

### CREATE TABLE 语法

```sql
CREATE TABLE books (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    author_id INTEGER,
    year INTEGER,
    rating REAL,
    FOREIGN KEY (author_id) REFERENCES authors(id)
);
```

### SQLite 常用数据类型

| 类型 | 存储内容 | 例子 |
|------|---------|------|
| `INTEGER` | 整数 | 42, 2023, -5 |
| `REAL` | 浮点数 | 4.5, 3.14159 |
| `TEXT` | 字符串 | 'Hello', "World" |
| `BLOB` | 二进制数据 | 图片、文件 |
| `NUMERIC` | 精确数值 | 用于日期/时间/布尔 |

> 💡 SQLite 的类型系统是**灵活类型**（Manifest Typing）：列的类型只是建议，实际上可以存任何类型。这和 MySQL/PostgreSQL 的严格类型不同。

### 选择数据类型的原则

```
整数 → INTEGER       (年龄、数量、年份)
小数 → REAL           (评分、金额)
文本 → TEXT           (名字、描述、地址)
日期 → TEXT (ISO8601) 或 INTEGER (Unix 时间戳)
```

---

## 4.5 约束（3:56:53）

约束是建表时定义的**规则**，数据库自动强制执行。

| 约束 | 效果 | 例子 |
|------|------|------|
| `PRIMARY KEY` | 唯一 + 非空，标识每一行 | `id INTEGER PRIMARY KEY` |
| `NOT NULL` | 不能为空 | `title TEXT NOT NULL` |
| `UNIQUE` | 值不能重复 | `email TEXT UNIQUE` |
| `DEFAULT` | 不指定值时用默认值 | `year INTEGER DEFAULT 2023` |
| `CHECK` | 值必须满足条件 | `CHECK (rating >= 0 AND rating <= 5)` |
| `FOREIGN KEY` | 值必须在另一张表中存在 | `FOREIGN KEY (a_id) REFERENCES authors(id)` |

```sql
CREATE TABLE books (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    year INTEGER DEFAULT 2024,
    rating REAL CHECK (rating >= 0 AND rating <= 5),
    isbn TEXT UNIQUE,
    author_id INTEGER NOT NULL,
    FOREIGN KEY (author_id) REFERENCES authors(id)
);
```

---

## 4.6 修改表结构（4:07:26）

### ALTER TABLE

```sql
-- 添加列
ALTER TABLE books ADD COLUMN language TEXT DEFAULT 'English';

-- 删除列（SQLite 3.35+）
ALTER TABLE books DROP COLUMN language;

-- 重命名列（SQLite 3.25+）
ALTER TABLE books RENAME COLUMN title TO book_title;

-- 重命名表
ALTER TABLE books RENAME TO books_2024;
```

> ⚠️ SQLite 的 ALTER TABLE 功能有限。不支持修改列类型、不支持添加约束到已有列。复杂修改需要重建表。

---

## 4.7 外键约束深入（4:20:30）

```sql
CREATE TABLE books (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    author_id INTEGER NOT NULL,
    FOREIGN KEY (author_id) REFERENCES authors(id)
        ON DELETE CASCADE     -- 作者删了 → 书也自动删
        ON UPDATE CASCADE     -- 作者 id 变了 → 跟着变
);
```

### ON DELETE / ON UPDATE 行为

| 行为 | DELETE 效果 | UPDATE 效果 |
|------|------------|------------|
| `CASCADE` | 父删子也删 | 父改子也改 |
| `SET NULL` | 子 FK 变 NULL | 子 FK 变 NULL |
| `SET DEFAULT` | 子 FK 变默认值 | 子 FK 变默认值 |
| `RESTRICT` | 有子就阻止删 | 有子就阻止改 |
| `NO ACTION` | 同 RESTRICT | 同 RESTRICT |

---

## 4.8 插入数据（4:26:31）

```sql
-- 插入单行
INSERT INTO authors (name, birth_year)
VALUES ('Stephen King', 1947);

-- 插入多行
INSERT INTO authors (name, birth_year) VALUES
    ('Andy Weir', 1972),
    ('Gillian Flynn', 1971),
    ('James Clear', 1986);

-- 不指定列名（按表定义顺序提供所有值）
INSERT INTO authors VALUES (NULL, 'New Author', 1990);

-- 默认值自动填充
INSERT INTO authors (name) VALUES ('Unknown Author');
-- birth_year 使用 DEFAULT 值（如果有的话），否则 NULL
```

---

## 4.9 导入 CSV（4:46:57）

```sql
-- SQLite 命令行方式
.import data.csv books --skip 1

-- 或用 Python/sqlite3
-- import csv; cursor.executemany('INSERT ...', rows)
```

---

## 4.10 删除数据（5:03:27）

```sql
-- 删除特定行
DELETE FROM books WHERE year < 1900;

-- 删除所有行
DELETE FROM books;

-- 删除与外键的交互：
-- 如果 books 有 ON DELETE CASCADE，删除作者会级联删除书
-- 如果 books 有 ON DELETE RESTRICT，有书时不能删作者
```

### 安全删除流程

1. 先用 SELECT 确认要删的行：`SELECT * FROM books WHERE year < 1900;`
2. 确认无误再用 DELETE：`DELETE FROM books WHERE year < 1900;`
3. 批量删除前考虑备份或事务

---

## 4.11 更新数据（5:25:20）

```sql
-- 更新单列
UPDATE books SET rating = 5.0 WHERE id = 1;

-- 更新多列
UPDATE books SET rating = 4.5, year = 2023 WHERE id = 1;

-- 批量更新
UPDATE books SET genre = 'Classic' WHERE year < 1950;

-- 基于其他表更新（子查询）
UPDATE books SET author_id = (
    SELECT id FROM authors WHERE name = 'Stephen King'
)
WHERE title = 'The Shining';
```

> ⚠️ 不带 WHERE 的 UPDATE/DELETE 会影响**所有行**！养成「先 SELECT，再 UPDATE/DELETE」的习惯。

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->



---

## 📝 本章速查

```sql
-- ==================== 分组 ====================
SELECT genre, COUNT(*), AVG(rating)
FROM books
WHERE year >= 2000
GROUP BY genre
HAVING COUNT(*) >= 2
ORDER BY AVG(rating) DESC;

-- ==================== 建表 ====================
CREATE TABLE books (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    year INTEGER DEFAULT 2024,
    rating REAL CHECK (rating BETWEEN 0 AND 5),
    isbn TEXT UNIQUE,
    author_id INTEGER NOT NULL,
    FOREIGN KEY (author_id) REFERENCES authors(id) ON DELETE CASCADE
);

-- ==================== 修改表 ====================
ALTER TABLE books ADD COLUMN language TEXT;
ALTER TABLE books RENAME COLUMN title TO name;
ALTER TABLE books DROP COLUMN language;

-- ==================== 插入 ====================
INSERT INTO books (title, year) VALUES ('New Book', 2024);
INSERT INTO books (title, year) VALUES ('A', 2020), ('B', 2021);

-- ==================== 更新 ====================
UPDATE books SET rating = 5.0 WHERE id = 1;
UPDATE books SET rating = 4.0, year = 2023 WHERE genre = 'Fiction';

-- ==================== 删除 ====================
DELETE FROM books WHERE year < 1900;
DELETE FROM books;  -- 删所有行！
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
| 2 | | |
