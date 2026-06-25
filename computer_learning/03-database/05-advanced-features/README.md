# 05 - 高级特性（Advanced Features）

> 📺 课程时间：5:49:09 – 7:31:17

---

## 知识点清单

### 5.1 触发器（5:49:09）
- [ ] `CREATE TRIGGER` 语法
- [ ] `BEFORE` / `AFTER` / `INSTEAD OF` 触发时机
- [ ] `INSERT` / `UPDATE` / `DELETE` 触发事件
- [ ] `OLD` 和 `NEW` 引用

### 5.2 软删除（6:02:25）
- [ ] 软删除 vs 硬删除
- [ ] `deleted_at` / `is_deleted` 字段设计
- [ ] 触发器实现自动软删除
- [ ] 查询时过滤已删除记录

### 5.3 多对多关系实现（6:09:07）
- [ ] 联结表（Junction Table / Join Table）
- [ ] 多对多关系的查询
- [ ] 多对多关系中的附加属性

### 5.4 VIEW 视图（6:22:07）
- [ ] `CREATE VIEW` 创建视图
- [ ] 视图的本质：存储的查询
- [ ] 通过视图简化复杂查询
- [ ] 视图的更新限制

### 5.5 分组聚合进阶（6:31:11）
- [ ] 复杂的分组查询
- [ ] 多层聚合
- [ ] 聚合结果的使用

### 5.6 CTE 公共表表达式（6:44:24）
- [ ] `WITH ... AS` 语法
- [ ] CTE vs 子查询 vs 临时表
- [ ] 递归 CTE
- [ ] CTE 的可读性优势

### 5.7 数据匿名化（7:02:14）
- [ ] 使用 VIEW 隐藏敏感字段
- [ ] 数据脱敏策略
- [ ] 安全数据共享

### 5.8 触发器与视图（7:15:24）
- [ ] `INSTEAD OF` 触发器
- [ ] 通过触发器让视图可更新
- [ ] 视图 + 触发器的组合应用

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->

### CTE 语法速查

```sql
WITH cte_name AS (
    SELECT ... FROM ...
)
SELECT * FROM cte_name WHERE ...;
```

### 软删除实现思路

```sql
-- 表中添加 deleted_at 列
ALTER TABLE items ADD COLUMN deleted_at DATETIME;

-- 查询时过滤
SELECT * FROM items WHERE deleted_at IS NULL;

-- 触发器实现软删除
CREATE TRIGGER soft_delete
INSTEAD OF DELETE ON items_view
BEGIN
    UPDATE items SET deleted_at = DATETIME('now') WHERE id = OLD.id;
END;
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
