# 03 - 高级查询（Advanced Queries）

> 📺 课程时间：1:49:13 – 2:37:37
> ✅ 学习状态：进行中

---

## 3.1 子查询（1:49:13）

### 什么是子查询？

**子查询（Subquery）= 嵌套在另一个 SQL 语句内部的 SELECT 语句。**

```sql
-- 子查询在括号里
SELECT title, rating
FROM books
WHERE rating > (SELECT AVG(rating) FROM books);
--               ↑ 子查询先执行，得出平均评分 4.19
-- 外层查询再找评分 > 4.19 的书
```

### 子查询的执行顺序

```
1. 先执行 () 里的子查询，得到一个值（或结果集）
2. 把结果代入外层查询
3. 执行外层查询

    外层: SELECT title FROM books WHERE rating > ( 内层 )
                                                    ↓
    内层:                                         SELECT AVG(rating) FROM books
                                                    ↓
                                                 返回 4.19
                                                    ↓
    外层变成: SELECT title FROM books WHERE rating > 4.19
```

### 子查询能放在哪里

```sql
-- ① WHERE 子句中（最常用）
SELECT title FROM books
WHERE author_id = (SELECT id FROM authors WHERE name = 'Stephen King');

-- ② SELECT 子句中（作为计算列）
SELECT title, rating,
       (SELECT ROUND(AVG(rating), 2) FROM books) AS overall_avg
FROM books;
-- 每行都显示「本书评分 + 整体平均分」

-- ③ FROM 子句中（子查询当临时表，必须给别名）
SELECT avg_rating
FROM (SELECT genre, AVG(rating) AS avg_rating FROM books GROUP BY genre) AS genre_avgs
WHERE avg_rating > 4.0;
```

---

## 3.2 嵌套查询进阶（2:00:13）

### 子查询返回单一值 vs 多值

```sql
-- 返回单一值：用比较运算符
SELECT title FROM books
WHERE pages > (SELECT AVG(pages) FROM books);

-- 返回多值（一列多行）：用 IN
SELECT title FROM books
WHERE author_id IN (
    SELECT id FROM authors WHERE birth_year > 1970
);
```

### 相关子查询（Correlated Subquery）

普通子查询独立执行一次；**相关子查询依赖于外层的每一行，对每一行都执行一次。**

```sql
-- 相关子查询：找出「比自己类型平均评分高」的书
SELECT title, genre, rating
FROM books AS b1
WHERE rating > (
    SELECT AVG(rating)
    FROM books AS b2
    WHERE b2.genre = b1.genre  -- ← 引用了外层 b1.genre！
);
-- 对每本书，子查询计算「和它同类型的书的平均分」
```

> ⚠️ 相关子查询性能较差——外层有 1000 行，子查询就执行 1000 次。

---

## 3.3 IN 关键字（2:08:39）

`IN` 结合子查询是非常常见的模式。

```sql
-- 找出「1970 年后出生的作者写的书」
SELECT title FROM books
WHERE author_id IN (
    SELECT id FROM authors WHERE birth_year > 1970
);

-- 等价于（但如果子查询结果很多，IN 比 JOIN 可读性好）
SELECT b.title FROM books b
JOIN authors a ON b.author_id = a.id
WHERE a.birth_year > 1970;
```

### NOT IN

```sql
-- 找出「没有写过书的作者」
SELECT name FROM authors
WHERE id NOT IN (
    SELECT DISTINCT author_id FROM books WHERE author_id IS NOT NULL
);
```

---

## 3.4 JOIN 连接（2:19:06 – 2:27:26）

### 为什么需要 JOIN？

数据存在多张表里，但查询时往往需要**跨表取数据**。JOIN 就是表与表之间的粘合剂。

```
authors                     books
┌────┬──────────────┐      ┌────┬──────────────┬───────────┐
│ id │ name         │      │ id │ title        │ author_id │
├────┼──────────────┤      ├────┼──────────────┼───────────┤
│ 1  │ Stephen King │      │ 1  │ The Shining  │ 1         │
│ 2  │ Andy Weir    │      │ 2  │ It           │ 1         │
│ 3  │ F. Herbert   │      │ 3  │ The Martian  │ 2         │
└────┴──────────────┘      └────┴──────────────┴───────────┘

INNER JOIN ... ON authors.id = books.author_id
                     ↓
┌──────────────┬──────────────┐
│ name         │ title        │
├──────────────┼──────────────┤
│ Stephen King │ The Shining  │
│ Stephen King │ It           │
│ Andy Weir    │ The Martian  │
└──────────────┴──────────────┘
```

### JOIN 通用语法

```sql
SELECT 列1, 列2, ...
FROM 表1
JOIN 表2 ON 表1.某列 = 表2.某列;
--  ON 后面是连接条件（通常是 PK = FK）
```

### 表别名 — 给表起小名

```sql
SELECT b.title, a.name, p.name
FROM books AS b            -- b = books 的别名
JOIN authors AS a ON b.author_id = a.id    -- a = authors 的别名
JOIN publishers AS p ON b.publisher_id = p.id;
-- AS 可以省略：FROM books b
```

> 💡 多表 JOIN 时，表别名能让 SQL 短一半，必须掌握。

---

## 3.5 JOIN 类型（2:27:26）

### 四种 JOIN 的区别

用两个小表做例子：

```
表 A（authors）       表 B（books）
┌────┬──────────┐    ┌────┬──────────────┬────┐
│ id │ name     │    │ id │ title        │a_id│
├────┼──────────┤    ├────┼──────────────┼────┤
│ 1  │ King     │    │ 1  │ The Shining  │ 1  │
│ 2  │ Weir     │    │ 2  │ The Martian  │ 2  │
│ 3  │ Orwell   │    │ 3  │ Dune         │ 5  │ ← a_id=5 在 A 里不存在
└────┴──────────┘    └────┴──────────────┴────┘
```

```
INNER JOIN (A.id = B.a_id)        LEFT JOIN (A LEFT JOIN B)
返回两表匹配的行                           左表全部保留，右表无匹配填 NULL
┌──────┬──────────────┐          ┌──────┬──────────────┐
│ name │ title        │          │ name │ title        │
├──────┼──────────────┤          ├──────┼──────────────┤
│ King │ The Shining  │          │ King │ The Shining  │
│ Weir │ The Martian  │          │ Weir │ The Martian  │
└──────┴──────────────┘          │Orwell│ NULL         │ ← 没书，保留
      Dune 丢了（a_id=5不存在）    └──────┴──────────────┘
```

```
RIGHT JOIN (A RIGHT JOIN B)       FULL JOIN（SQLite 不支持，需模拟）
右表全部保留，左表无匹配填 NULL          两表全部保留
┌──────┬──────────────┐          ┌──────┬──────────────┐
│ name │ title        │          │ King │ The Shining  │
├──────┼──────────────┤          │ Weir │ The Martian  │
│ King │ The Shining  │          │Orwell│ NULL         │
│ Weir │ The Martian  │          │ NULL │ Dune         │
│ NULL │ Dune         │← 保留    └──────┴──────────────┘
└──────┴──────────────┘
```

### 各类型速查

| JOIN 类型 | 保留什么 | 使用场景 |
|-----------|----------|----------|
| `INNER JOIN` | 两表都有的 | **最常用**，找匹配数据 |
| `LEFT JOIN` | 左表全部 | 「所有作者 + 他们的书（没书的也列出来）」 |
| `RIGHT JOIN` | 右表全部 | 同上但反过来（很少用，换成 LEFT 即可） |
| `FULL JOIN` | 两表全部 | 找两边的差异 |

```sql
-- INNER JOIN（默认，最常用）
SELECT a.name, b.title
FROM authors a
INNER JOIN books b ON a.id = b.author_id;

-- LEFT JOIN：保留左表全部
SELECT a.name, b.title
FROM authors a
LEFT JOIN books b ON a.id = b.author_id;
-- 没书的作者也会出现，title 显示 NULL

-- 多表 JOIN
SELECT b.title, a.name AS author, p.name AS publisher
FROM books b
JOIN authors a ON b.author_id = a.id
JOIN publishers p ON b.publisher_id = p.id;
```

---

## 3.6 集合操作（2:37:37）

集合操作把**两个查询的结果集**当集合来运算。

### UNION — 合并（去重）

```sql
-- 把两个查询结果摞在一起，自动去重
SELECT title FROM books WHERE year < 2000
UNION
SELECT title FROM books WHERE genre = 'Science Fiction';
-- 同时满足两个条件的也只出现一次

-- UNION ALL：保留重复行（更快，因为不用去重）
SELECT title FROM books WHERE year < 2000
UNION ALL
SELECT title FROM books WHERE genre = 'Science Fiction';
```

### INTERSECT — 交集

```sql
-- 同时满足两个条件的行
SELECT title FROM books WHERE year < 2000
INTERSECT
SELECT title FROM books WHERE genre = 'Science Fiction';
-- 2000 年之前出版的科幻小说
```

### EXCEPT — 差集

```sql
-- 在第一个查询中但不在第二个查询中的行
SELECT title FROM books WHERE year < 2000
EXCEPT
SELECT title FROM books WHERE genre = 'Science Fiction';
-- 2000 年之前的书，但不是科幻小说
```

### 集合操作的规则

1. 两个 SELECT 必须有**相同的列数**
2. 对应列的数据类型必须兼容
3. 列名以第一个 SELECT 的为准
4. `UNION` 去重，`UNION ALL` 不去重（更快）

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->



---

## 📝 本章速查

```sql
-- ==================== 子查询 ====================
-- WHERE 中的子查询（最常用）
SELECT * FROM books WHERE rating > (SELECT AVG(rating) FROM books);
SELECT * FROM books WHERE author_id = (SELECT id FROM authors WHERE name = '...');

-- 多值子查询 + IN
SELECT * FROM books WHERE author_id IN (SELECT id FROM authors WHERE birth_year > 1970);

-- SELECT 中的子查询（计算列）
SELECT title, rating, (SELECT AVG(rating) FROM books) AS avg_all FROM books;

-- FROM 中的子查询（必须有别名）
SELECT * FROM (SELECT genre, AVG(rating) AS avg_r FROM books GROUP BY genre) AS t;

-- ==================== JOIN ====================
-- INNER JOIN：两表匹配的行
SELECT * FROM a INNER JOIN b ON a.id = b.a_id;

-- LEFT JOIN：左表全部 + 右表匹配
SELECT * FROM a LEFT JOIN b ON a.id = b.a_id;

-- 多表 JOIN
SELECT * FROM books b
JOIN authors a ON b.author_id = a.id
JOIN publishers p ON b.publisher_id = p.id;

-- ==================== 集合操作 ====================
-- UNION：合并（去重）
SELECT col1, col2 FROM t1 UNION SELECT col1, col2 FROM t2;

-- UNION ALL：合并（不去重，更快）
SELECT col1, col2 FROM t1 UNION ALL SELECT col1, col2 FROM t2;

-- INTERSECT：交集
SELECT col1, col2 FROM t1 INTERSECT SELECT col1, col2 FROM t2;

-- EXCEPT：差集（t1 有但 t2 没有的）
SELECT col1, col2 FROM t1 EXCEPT SELECT col1, col2 FROM t2;

-- ==================== 表别名 ====================
SELECT b.title, a.name
FROM books b          -- b = books
JOIN authors a        -- a = authors
ON b.author_id = a.id;
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
| 2 | | |
