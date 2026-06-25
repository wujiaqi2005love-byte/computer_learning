# 07 - 数据库系统与安全（Database Systems & Security）

> 📺 课程时间：9:01:45 – 结束（11:08:36）

---

## 知识点清单

### 7.1 SQLite / MySQL / PostgreSQL 对比（9:01:45）
- [ ] SQLite：嵌入式、零配置、单文件
- [ ] MySQL：Web 应用广泛、多种存储引擎
- [ ] PostgreSQL：功能最强、标准兼容最好
- [ ] 如何选择合适的数据库

### 7.2 MySQL 深入（9:10:08 – 9:56:00）
- [ ] MySQL 数据类型：
  - 字符串：`CHAR`, `VARCHAR`, `TEXT`
  - 数值：`INT`, `DECIMAL`, `FLOAT`
  - 日期：`DATE`, `DATETIME`, `TIMESTAMP`
- [ ] 约束与默认值
- [ ] MySQL vs SQLite 语法差异

### 7.3 存储过程（9:56:00 – 10:06:54）
- [ ] `CREATE PROCEDURE` 创建存储过程
- [ ] 参数传递（IN / OUT / INOUT）
- [ ] 存储过程 vs 应用层代码
- [ ] 事务在存储过程中的使用

### 7.4 PostgreSQL 简介（10:15:53）
- [ ] PostgreSQL 特性概览
- [ ] 与 MySQL 的数据类型和语法对比（10:26:56）
- [ ] 高级功能：JSON 支持、全文搜索、地理数据

### 7.5 可扩展性架构（10:33:19）
- [ ] 复制（Replication）
  - 主从复制（Master-Slave）
  - 同步 vs 异步复制（10:39:44）
- [ ] 分片（Sharding）
  - 水平分片 vs 垂直分片
- [ ] 读写分离
- [ ] CAP 理论简介

### 7.6 用户与权限管理（10:47:59）
- [ ] `CREATE USER` / `GRANT` / `REVOKE`
- [ ] 权限粒度：数据库级、表级、列级
- [ ] 最小权限原则

### 7.7 SQL 注入（10:55:01）
- [ ] SQL 注入的原理
- [ ] 经典注入示例
- [ ] 危害：数据泄露、数据篡改、删库

### 7.8 防御措施（11:01:46）
- [ ] 预编译语句（Prepared Statements）⭐ 首选防御
- [ ] 输入验证与转义
- [ ] 参数化查询
- [ ] ORM 框架的保护
- [ ] 最小权限原则

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->

### 数据库选型速查

| 场景 | 推荐 |
|------|------|
| 移动 App / 桌面软件 / 原型 | SQLite |
| Web 应用 / CMS / 博客 | MySQL |
| 数据分析 / 复杂查询 / 企业应用 | PostgreSQL |

### SQL 注入防御

```python
# ❌ 危险：字符串拼接
cursor.execute(f"SELECT * FROM users WHERE name = '{user_input}'")

# ✅ 安全：参数化查询
cursor.execute("SELECT * FROM users WHERE name = ?", (user_input,))
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
