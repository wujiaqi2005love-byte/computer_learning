# 05 - 高级特性（Advanced Features）

> 📺 课程时间：5:49:09 – 7:31:17
> ✅ 学习状态：进行中

---

## 5.1 触发器（Triggers）

### 什么是触发器？

**触发器（Trigger）= 当某个表上发生 INSERT / UPDATE / DELETE 时，自动执行的一段 SQL 代码。**

就像一个"事件监听器"——数据库检测到特定操作，自动触发预设动作。

```sql
CREATE TRIGGER 触发器名
BEFORE/AFTER INSERT/UPDATE/DELETE ON 表名
FOR EACH ROW
BEGIN
    -- 自动执行的 SQL 语句
END;
```

### 触发时机

| 关键字 | 含义 |
|--------|------|
| `BEFORE` | 在操作执行**之前**触发（可以修改 NEW 值） |
| `AFTER` | 在操作执行**之后**触发（常用于记录日志） |
| `INSTEAD OF` | 替代操作（主要用于 VIEW，让视图可更新） |

### OLD 和 NEW 引用

在触发器内部，可以访问操作前后的行数据：

| | INSERT | UPDATE | DELETE |
|---|--------|--------|--------|
| `NEW.列名` | ✅ 新插入的值 | ✅ 更新后的值 | ❌ 不可用 |
| `OLD.列名` | ❌ 不可用 | ✅ 更新前的值 | ✅ 被删行的值 |

### 常见用例

**① 自动更新时间戳**

```sql
CREATE TRIGGER update_timestamp
AFTER UPDATE ON courses
FOR EACH ROW
BEGIN
    UPDATE courses SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

**② 审计日志——记录每次数据变更**

```sql
CREATE TRIGGER log_student_changes
AFTER UPDATE ON students
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, action, record_id, old_data, new_data, changed_at)
    VALUES ('students', 'UPDATE', NEW.id, OLD.name, NEW.name, datetime('now'));
END;
```

**③ 数据验证——阻止非法数据**

```sql
CREATE TRIGGER validate_grade
BEFORE INSERT ON enrollments
FOR EACH ROW
BEGIN
    SELECT CASE
        WHEN NEW.grade < 0 OR NEW.grade > 100 THEN
            RAISE(ABORT, '成绩必须在 0-100 之间')
    END;
END;
```

**④ 防止误删——有选课记录时不删课**

```sql
CREATE TRIGGER prevent_course_delete
BEFORE DELETE ON courses
FOR EACH ROW
WHEN (SELECT COUNT(*) FROM enrollments WHERE course_id = OLD.id) > 0
BEGIN
    SELECT RAISE(ABORT, '该课程还有学生选修，不能删除');
END;
```

### 触发器 vs 应用层代码

| | 触发器 | 应用层代码 |
|---|--------|-----------|
| 执行位置 | 数据库内部 | 应用服务器 |
| 网络开销 | 无（数据库内执行） | 每次额外一次数据库调用 |
| 一定会执行？ | ✅ 无论谁来操作都触发 | ❌ 绕开应用层直接操作数据库则不触发 |
| 可维护性 | 隐蔽，容易被忘记 | 显式，便于追踪调试 |
| 适用场景 | 审计日志、数据验证、自动时间戳 | 复杂业务逻辑、需要外部 API 的操作 |

> 💡 **原则**：触发器适合"数据完整性"逻辑（日志、验证、时间戳）。业务逻辑放应用层，保持数据库简单可预测。

### 删除触发器

```sql
DROP TRIGGER IF EXISTS trigger_name;
```

---

## 5.2 软删除（Soft Deletes）

### 硬删除 vs 软删除

```
硬删除（Hard Delete）：DELETE FROM ... WHERE ...
  → 数据从磁盘永久消失
  → 无法恢复
  → 外键 CASCADE 可能级联删除大量数据

软删除（Soft Delete）：UPDATE ... SET deleted_at = datetime('now') WHERE ...
  → 数据还在表中，只是标记为"已删除"
  → 可以随时恢复（undo）
  → 查询时需要加 WHERE deleted_at IS NULL
```

### 实现方式

```sql
-- 方式 1：deleted_at 时间戳（推荐，能知道什么时候删的）
CREATE TABLE courses (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    deleted_at TEXT DEFAULT NULL  -- NULL = 未删，有值 = 已删
);

-- 方式 2：is_active 布尔标记
CREATE TABLE courses (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    is_active INTEGER DEFAULT 1  -- 1 = 活跃，0 = 已删
);
```

### 软删除操作

```sql
-- "删除"（其实是更新）
UPDATE courses SET deleted_at = datetime('now') WHERE id = 3;

-- "恢复"
UPDATE courses SET deleted_at = NULL WHERE id = 3;

-- 彻底删除（真正删除已标记的数据）
DELETE FROM courses WHERE deleted_at IS NOT NULL
    AND deleted_at < datetime('now', '-30 days');  -- 30 天前删的
```

### 软删除 + 视图 = 最佳实践

```sql
-- 创建视图自动过滤已删除数据
CREATE VIEW active_courses AS
SELECT id, name, credits, teacher_id
FROM courses
WHERE deleted_at IS NULL;

-- 应用层只查视图，永远看不到已删除的课
SELECT * FROM active_courses;
```

### 软删除的利弊

| | 软删除 | 硬删除 |
|---|--------|--------|
| 数据恢复 | ✅ 一行 UPDATE 就恢复 | ❌ 只能从备份恢复 |
| 审计追溯 | ✅ 保留历史操作痕迹 | ❌ 数据丢了无迹可寻 |
| 查询复杂度 | ⚠️ 每次要加 WHERE 条件 | ✅ 不需要 |
| 存储空间 | ❌ 数据不释放，表越来越大 | ✅ 释放空间 |
| 唯一约束 | ❌ 同名已删记录也冲突（需特殊处理） | ✅ 简单 |
| 适用场景 | 用户数据、订单、内容、课程 | 临时数据、缓存、日志 |

---

## 5.3 多对多关系实现（Many-to-Many）

### 问题

一个学生选多门课，一门课有多个学生。这就是 **N:M 关系**。

```
students                     courses
┌────┬──────┐               ┌────┬──────────────┐
│ id │ name │               │ id │ name         │
├────┼──────┤               ├────┼──────────────┤
│ 1  │ 张三 │               │ 1  │ Python 入门   │
│ 2  │ 李四 │               │ 2  │ 数据库基础    │
│ 3  │ 王五 │               │ 3  │ 机器学习      │
└────┴──────┘               └────┴──────────────┘

张三 → Python, 数据库, 机器学习
李四 → Python, 机器学习
王五 → 数据库

❌ 单在 students 加 FK → 只能表示一门课
❌ 单在 courses 加 FK → 只能表示一个学生
```

### 解决方案：联结表

```
enrollments（联结表 / Junction Table）
┌────┬────────────┬───────────┬───────┬──────────────┐
│ id │ student_id │ course_id │ grade │ enrolled_at  │
├────┼────────────┼───────────┼───────┼──────────────┤
│ 1  │ 1 (张三)   │ 1 (Python)│ 92.5  │ 2024-01-15   │
│ 2  │ 1 (张三)   │ 2 (DB)    │ 88.0  │ 2024-01-15   │
│ 3  │ 1 (张三)   │ 3 (ML)    │ 95.0  │ 2024-01-16   │
│ 4  │ 2 (李四)   │ 1 (Python)│ 85.5  │ 2024-01-15   │
│ 5  │ 2 (李四)   │ 3 (ML)    │ 91.0  │ 2024-01-17   │
│ 6  │ 3 (王五)   │ 2 (DB)    │ 79.0  │ 2024-01-16   │
└────┴────────────┴───────────┴───────┴──────────────┘
```

**联结表的核心价值**：不仅解决 N:M，还能存储关系的附加信息（成绩 grade、选课日期 enrolled_at）。

### 联结表设计要点

```sql
CREATE TABLE enrollments (
    id INTEGER PRIMARY KEY,
    student_id INTEGER NOT NULL,
    course_id INTEGER NOT NULL,
    grade REAL CHECK (grade >= 0 AND grade <= 100),
    enrolled_at TEXT DEFAULT (date('now')),
    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE,
    UNIQUE(student_id, course_id)  -- 防止重复选课
);
```

关键约束：
- **两个 FK** 分别指向两张父表
- **UNIQUE(student_id, course_id)** 防止同一学生重复选同一门课
- **ON DELETE CASCADE** 学生删了 → 选课记录自动删；课程删了同理
- **额外属性**（grade, enrolled_at）描述这个关系的细节

### 多对多查询

```sql
-- 查张三的所有课和成绩
SELECT c.name, e.grade
FROM students s
JOIN enrollments e ON s.id = e.student_id
JOIN courses c ON e.course_id = c.id
WHERE s.name = '张三';

-- 每门课有多少人选（聚合）
SELECT c.name, COUNT(e.student_id) AS student_count,
       ROUND(AVG(e.grade), 1) AS avg_grade
FROM courses c
LEFT JOIN enrollments e ON c.id = e.course_id
GROUP BY c.id;

-- 找出没选任何课的学生
SELECT name FROM students
WHERE id NOT IN (SELECT DISTINCT student_id FROM enrollments);

-- 找出同时选了 Python 和 数据库 的学生
SELECT s.name
FROM students s
JOIN enrollments e1 ON s.id = e1.student_id
JOIN enrollments e2 ON s.id = e2.student_id
JOIN courses c1 ON e1.course_id = c1.id
JOIN courses c2 ON e2.course_id = c2.id
WHERE c1.name = 'Python 入门' AND c2.name = '数据库基础';
```

### 联结表命名惯例

| 风格 | 示例 |
|------|------|
| 动词/名词（推荐） | `enrollments`, `purchases`, `subscriptions` |
| 表名拼接 | `student_courses`, `book_authors` |
| 动作词组 | `course_registrations`, `order_items` |

---

## 5.4 视图（Views）

### 什么是视图？

**视图（View）= 一个被保存的 SELECT 查询，可以像表一样使用。**

视图**不存储数据**——它只是一个"窗口"，每次访问时动态执行背后的查询。

```
┌─────────────────────────────────┐
│   VIEW: course_summary          │  ← 虚拟表，不存数据
│   ┌─────────────────────────┐   │
│   │ SELECT c.name,          │   │
│   │   COUNT(e.student_id)   │   │
│   │ FROM courses c          │   │
│   │ LEFT JOIN enrollments e │   │
│   │ GROUP BY c.name         │   │
│   └─────────────────────────┘   │
└─────────────────────────────────┘
         ↑ 每次查 VIEW 时动态执行底层查询
```

### 创建和使用

```sql
-- 创建：把复杂查询封装起来
CREATE VIEW student_transcript AS
SELECT s.name AS student, c.name AS course, e.grade,
       CASE
           WHEN e.grade >= 90 THEN 'A'
           WHEN e.grade >= 80 THEN 'B'
           WHEN e.grade >= 70 THEN 'C'
           WHEN e.grade >= 60 THEN 'D'
           ELSE 'F'
       END AS letter_grade
FROM students s
JOIN enrollments e ON s.id = e.student_id
JOIN courses c ON e.course_id = c.id;

-- 使用：和普通表一模一样
SELECT * FROM student_transcript WHERE student = '张三';
SELECT course, AVG(grade) FROM student_transcript GROUP BY course;
```

### 为什么用视图？

| 好处 | 说明 |
|------|------|
| **简化复杂查询** | 多表 JOIN + 聚合 + CASE 封装成一张"表" |
| **安全/权限控制** | 只暴露视图给用户，隐藏敏感列（如邮箱、密码） |
| **逻辑抽象** | 底层表结构改变，只需更新视图定义 |
| **可读性** | 给复杂查询起个有意义的名字 |

### 视图的限制

- **不是所有视图都能更新**：含 JOIN、聚合、DISTINCT、GROUP BY 的视图是只读的
- **不存数据**：每次查询都重新执行底层 SQL（某些数据库有物化视图）
- SQLite 中 `ALTER VIEW` 不支持 → 需 `DROP` 后重建

### INSTEAD OF 触发器——让视图可更新

```sql
-- 在视图上的 INSTEAD OF 触发器
CREATE TRIGGER delete_via_view
INSTEAD OF DELETE ON active_courses
FOR EACH ROW
BEGIN
    UPDATE courses SET deleted_at = datetime('now') WHERE id = OLD.id;
END;

-- 现在 DELETE FROM active_courses 会触发软删除，而不是报错
```

---

## 5.5 CTE（公共表表达式）

### 什么是 CTE？

**CTE（Common Table Expression）= 用 `WITH` 关键字定义的临时命名结果集，只在一条 SQL 语句内有效。**

```sql
WITH 临时名 AS (
    SELECT ...  -- 子查询
)
SELECT ... FROM 临时名 ...;
```

### 为什么用 CTE？

```sql
-- ❌ 嵌套子查询——从里往外读
SELECT ...
FROM (
    SELECT ...
    FROM (
        SELECT ... FROM ...
    )
);

-- ✅ CTE——从上往下读，像写程序
WITH
step1 AS (SELECT ... FROM ...),
step2 AS (SELECT ... FROM step1),
step3 AS (SELECT ... FROM step2)
SELECT ... FROM step3;
```

### 非递归 CTE

```sql
-- 找出平均分最高的课程
WITH course_avg AS (
    SELECT course_id, ROUND(AVG(grade), 1) AS avg_grade
    FROM enrollments
    GROUP BY course_id
)
SELECT c.name, ca.avg_grade
FROM courses c
JOIN course_avg ca ON c.id = ca.course_id
WHERE ca.avg_grade = (SELECT MAX(avg_grade) FROM course_avg);
```

### 多 CTE 链式组合

```sql
WITH
-- CTE 1：每门课的统计
course_stats AS (
    SELECT course_id, COUNT(*) AS students, AVG(grade) AS avg_g
    FROM enrollments
    GROUP BY course_id
),
-- CTE 2：只保留人气课（≥ 3 人选）
popular AS (
    SELECT * FROM course_stats WHERE students >= 3
)
-- 最终查询
SELECT c.name, p.students, p.avg_g
FROM courses c
JOIN popular p ON c.id = p.course_id
ORDER BY p.avg_g DESC;
```

### 递归 CTE

递归 CTE 可以调用自己——这是 CTE 最强大的用法。

```sql
-- 生成 1 到 10 的数字序列
WITH RECURSIVE counter(n) AS (
    SELECT 1                    -- ① 基础情况（种子）
    UNION ALL
    SELECT n + 1 FROM counter   -- ② 递归步骤（引用自身）
    WHERE n < 10                -- ③ 终止条件
)
SELECT * FROM counter;
-- 结果：1, 2, 3, 4, 5, 6, 7, 8, 9, 10
```

**递归 CTE 三步结构**：

```
WITH RECURSIVE name AS (
    基础查询（不引用自己）  ← 起点
    UNION ALL
    递归查询（引用 name）   ← 用上一步结果继续
    WHERE 终止条件          ← 防止无限递归
)
```

**实战：课程前置依赖链**

```sql
-- 找「算法设计」的所有前置课程链（前置课的前置课的前置课...）
WITH RECURSIVE prereq_chain(id, name, depth) AS (
    -- 基础：算法的直接前置课
    SELECT p.prereq_id, c.name, 1
    FROM prerequisites p
    JOIN courses c ON p.prereq_id = c.id
    WHERE p.course_id = (SELECT id FROM courses WHERE name = '算法设计')
    UNION ALL
    -- 递归：前置课的前置课
    SELECT p.prereq_id, c.name, pc.depth + 1
    FROM prerequisites p
    JOIN courses c ON p.prereq_id = c.id
    JOIN prereq_chain pc ON p.course_id = pc.id
    WHERE pc.depth < 10  -- 安全上限，防止循环依赖
)
SELECT * FROM prereq_chain ORDER BY depth;
```

### CTE vs 子查询 vs 视图 vs 临时表

| | CTE | 子查询 | VIEW | 临时表 |
|---|-----|--------|------|--------|
| 作用域 | 一条 SQL | 一条 SQL | 全局，持久化 | 当前连接 |
| 可复用 | 同一 SQL 内多次引用 | ❌ 写一次用一次 | ✅ 任意查询 | ✅ 当前连接内 |
| 递归支持 | ✅ | ❌ | ❌ | ❌ |
| 存数据 | ❌ | ❌ | ❌ | ✅ |
| 最佳场景 | 多步计算，递归 | 简单嵌套 | 频繁复用的封装 | 多次用到的中间结果 |

---

## 5.6 INSTEAD OF 触发器 + 视图组合

这是 VIEW 和 TRIGGER 的组合技——让只读视图变得可更新。

```sql
-- 场景：active_courses 视图 + 软删除
-- 直接在视图上 DELETE 会报错 → 用 INSTEAD OF 拦截

CREATE TRIGGER soft_delete_course
INSTEAD OF DELETE ON active_courses
FOR EACH ROW
BEGIN
    UPDATE courses
    SET deleted_at = datetime('now')
    WHERE id = OLD.id;
END;

-- 现在可以用自然的 DELETE 语法操作视图
DELETE FROM active_courses WHERE id = 3;
-- ↑ 实际上执行了软删除，不会报错
```

---

## 5.7 数据匿名化

通过 VIEW 隐藏敏感字段，对外提供安全的"窗口"：

```sql
-- 学生表有 email（敏感），创建视图隐藏它
CREATE VIEW public_students AS
SELECT id, name, enrolled_year  -- 不暴露 email
FROM students;

-- 对外只给 public_students 的访问权限
-- 内部管理员查原始 students 表
```

---

## ✏️ 我的笔记

<!-- 在这里记录你的学习笔记 -->



---

## 📝 本章速查

```sql
-- ==================== 触发器 ====================
-- 自动更新时间戳
CREATE TRIGGER trg_updated_at
AFTER UPDATE ON t FOR EACH ROW
BEGIN
    UPDATE t SET updated_at = datetime('now') WHERE id = NEW.id;
END;

-- 审计日志
CREATE TRIGGER trg_audit
AFTER DELETE ON students FOR EACH ROW
BEGIN
    INSERT INTO audit_log (action, old_data, changed_at)
    VALUES ('DELETE', OLD.name, datetime('now'));
END;

-- 数据验证
CREATE TRIGGER trg_check
BEFORE INSERT ON enrollments FOR EACH ROW
BEGIN
    SELECT CASE
        WHEN NEW.grade < 0 THEN RAISE(ABORT, '成绩不能为负数')
    END;
END;

-- 删除触发器
DROP TRIGGER IF EXISTS trg_name;

-- ==================== 视图 ====================
CREATE VIEW active_courses AS
SELECT * FROM courses WHERE deleted_at IS NULL;

CREATE VIEW transcript AS
SELECT s.name, c.name AS course, e.grade
FROM students s
JOIN enrollments e ON s.id = e.student_id
JOIN courses c ON e.course_id = c.id;

DROP VIEW IF EXISTS view_name;

-- ==================== CTE ====================
-- 简单 CTE
WITH stats AS (
    SELECT course_id, AVG(grade) AS avg_g
    FROM enrollments GROUP BY course_id
)
SELECT c.name, s.avg_g FROM courses c JOIN stats s ON c.id = s.course_id;

-- 多 CTE
WITH
step1 AS (SELECT ...),
step2 AS (SELECT ... FROM step1)
SELECT ... FROM step2;

-- 递归 CTE
WITH RECURSIVE seq(n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1 FROM seq WHERE n < 10
)
SELECT * FROM seq;

-- ==================== 软删除 ====================
ALTER TABLE t ADD COLUMN deleted_at TEXT;
UPDATE t SET deleted_at = datetime('now') WHERE id = 3;     -- 删
UPDATE t SET deleted_at = NULL WHERE id = 3;                -- 恢复
SELECT * FROM t WHERE deleted_at IS NULL;                   -- 查活跃的

-- ==================== 多对多 ====================
CREATE TABLE enrollments (
    id INTEGER PRIMARY KEY,
    student_id INTEGER NOT NULL REFERENCES students(id),
    course_id INTEGER NOT NULL REFERENCES courses(id),
    grade REAL,
    UNIQUE(student_id, course_id)
);

SELECT s.name, c.name, e.grade
FROM students s
JOIN enrollments e ON s.id = e.student_id
JOIN courses c ON e.course_id = c.id;
```

---

## ❓ 疑问记录

| # | 问题 | 解答 |
|---|------|------|
| 1 | | |
| 2 | | |
