# 04 - Schema 设计与数据操作（Schema & Data Manipulation）

> 📺 课程时间：2:52:56 – 5:25:20

---

## 知识点清单

### 4.1 分组与过滤（2:52:56）
- [ ] `GROUP BY` 分组查询
- [ ] `HAVING` 过滤分组结果
- [ ] `WHERE` vs `HAVING` 的区别
- [ ] 多列分组

### 4.2 Schema 设计入门（3:04:06）
- [ ] 如何设计一个数据库 Schema
- [ ] 需求分析 → 实体识别 → 关系建模
- [ ] 设计原则：避免冗余、保证完整性

### 4.3 范式化（3:16:31）
- [ ] 第一范式（1NF）：原子性
- [ ] 第二范式（2NF）：消除部分依赖
- [ ] 第三范式（3NF）：消除传递依赖
- [ ] 范式化的权衡：性能 vs 规范化

### 4.4 建表与数据类型（3:24:46 – 3:44:14）
- [ ] `CREATE TABLE` 语法
- [ ] 常用数据类型：
  - `INTEGER`, `REAL`, `NUMERIC`
  - `TEXT`, `BLOB`
  - `DATE`, `DATETIME`
- [ ] 如何选择合适的数据类型
- [ ] 主键/外键定义语法

### 4.5 约束（3:56:53）
- [ ] `NOT NULL` — 非空约束
- [ ] `UNIQUE` — 唯一约束
- [ ] `PRIMARY KEY` — 主键约束
- [ ] `DEFAULT` — 默认值
- [ ] `CHECK` — 检查约束

### 4.6 修改表结构（4:07:26）
- [ ] `ALTER TABLE ... ADD COLUMN`
- [ ] `ALTER TABLE ... DROP COLUMN`
- [ ] `ALTER TABLE ... RENAME COLUMN`
- [ ] 修改表的注意事项

### 4.7 外键约束（4:20:30）
- [ ] `FOREIGN KEY ... REFERENCES`
- [ ] `ON DELETE` 行为：`CASCADE`, `SET NULL`, `RESTRICT`
- [ ] `ON UPDATE` 行为

### 4.8 插入数据（4:26:31）
- [ ] `INSERT INTO ... VALUES`
- [ ] 插入多行数据
- [ ] 从 CSV 导入数据（4:46:57）

### 4.9 删除数据（5:03:27）
- [ ] `DELETE FROM ... WHERE`
- [ ] 删除与外键约束的交互（5:11:55）
- [ ] 安全删除的注意事项

### 4.10 更新数据（5:25:20）
- [ ] `UPDATE ... SET ... WHERE`
- [ ] 批量更新
- [ ] 更新与外键约束

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->

### 范式化要点

```
1NF: 每列不可再分（原子值）
2NF: 非主键列完全依赖于主键
3NF: 非主键列不依赖于其他非主键列
```

### CRUD 速查

```sql
-- CREATE
INSERT INTO table (col1, col2) VALUES (val1, val2);

-- READ (在第一章学过了)
SELECT * FROM table WHERE condition;

-- UPDATE
UPDATE table SET col1 = new_val WHERE condition;

-- DELETE
DELETE FROM table WHERE condition;
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
