# 03 - 高级查询（Advanced Queries）

> 📺 课程时间：1:49:13 – 2:37:37

---

## 知识点清单

### 3.1 子查询（1:49:13）
- [ ] 什么是子查询（Subquery）
- [ ] 嵌套查询的执行顺序
- [ ] 在 `WHERE` 中使用子查询
- [ ] 在 `SELECT` 中使用子查询

### 3.2 嵌套查询进阶（2:00:13）
- [ ] 子查询与聚合函数的组合
- [ ] 相关子查询（Correlated Subquery）
- [ ] 子查询的性能考量

### 3.3 IN 关键字（2:08:39）
- [ ] `IN` + 子查询
- [ ] `NOT IN` 的用法
- [ ] 多条件子查询

### 3.4 JOIN 连接（2:19:06）
- [ ] 为什么需要 JOIN
- [ ] `INNER JOIN` — 取交集
- [ ] `LEFT JOIN` — 保留左表全部
- [ ] `RIGHT JOIN` — 保留右表全部
- [ ] `FULL JOIN` — 保留两表全部
- [ ] `ON` 条件 vs `WHERE` 条件的区别

### 3.5 集合操作（2:37:37）
- [ ] `UNION` — 合并结果集（去重）
- [ ] `UNION ALL` — 合并不去重
- [ ] `INTERSECT` — 取交集
- [ ] `EXCEPT` — 取差集
- [ ] 集合操作的列数/类型要求

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->

### JOIN 类型速查

```
INNER JOIN:  返回两表匹配的行
LEFT JOIN:   左表全部 + 右表匹配（无匹配填 NULL）
RIGHT JOIN:  右表全部 + 左表匹配（无匹配填 NULL）
FULL JOIN:   两表全部（无匹配填 NULL）
```

```sql
-- JOIN 示例
SELECT a.col1, b.col2
FROM table_a a
INNER JOIN table_b b ON a.id = b.a_id;
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
