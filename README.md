# 🖥️ Computer Science Fundamentals | 计算机科学基础自学仓库

> 中文说明
> 本仓库为从零起步自学计算机基础的完整学习记录，核心目标是夯实前置知识，为人工智能方向研究生阶段学习铺路。
> 所有笔记、习题、项目全部开源，欢迎同样自学 CS、准备转 AI 方向的同学一起参考交流。
>
> English Description
> This repository documents my self-study journey of fundamental computer science from scratch.
> The core goal is to build solid foundational knowledge to prepare for postgraduate study in artificial intelligence.
> All notes, exercises and mini-projects are fully open-source for self-learners who want to master CS & AI.

---

## 📋 Learning Roadmap | 学习路线图
### Core CS & AI Preparatory Courses
| No. | Subject 课程主题 | Status 状态 | Introduction 说明 |
|:---:|:----------------|:-----------|:------------------|
| 01 | [Programming 编程基础](01-programming/) | ⬜ To Start 待开始 | Main language: Python 以 Python 为主 |
| 02 | [Data Structures & Algorithms 数据结构与算法](02-data-structures-and-algorithms/) | ⬜ To Start 待开始 | Core foundation of Computer Science CS 核心基石 |
| 03 | [Database 数据库](03-database/) | 🔥 In Progress 进行中 | Course: Harvard CS50 SQL 哈佛 CS50 数据库课程 |
| 04 | [Operating Systems 操作系统](04-operating-systems/) | ⬜ To Start 待开始 | Computer bottom-layer theory 计算机底层理论 |
| 05 | [Computer Networks 计算机网络](05-computer-networks/) | ⬜ To Start 待开始 | Network transmission & communication 网络传输与通信原理 |
| 06 | [Computer Architecture 计算机组成原理](06-computer-architecture/) | ⬜ To Start 待开始 | CPU, memory, bus & hardware logic 处理器、内存、总线硬件逻辑 |
| 07 | [Math for AI 人工智能数学基础](07-math-for-ai/) | ⬜ To Start 待开始 | Linear Algebra, Probability & Statistics, Calculus 线代/概率论/微积分 |
| 08 | [Machine Learning 机器学习](08-machine-learning/) | ⬜ To Start 待开始 | Start learning after finishing all basic CS courses 完成基础后开启学习 |
| 09 | [Git Version Control Git版本控制](09-git/) | 🔥 In Progress 进行中 | Git basics + remote collaborative operation Git基础 + 远程团队协作实操 |

---

## 📖 Current Progress: Database | 当前学习进度：数据库
### Course: Harvard CS50's Intro to Databases with SQL
#### Chapter Schedule & Notes Directory 课程章节与笔记目录
| Chapter No. | Content 章节内容 | Course Timestamp 课程时间区间 |
|:---:|:---------------------|:------------------|
| [01 - SQL Fundamentals SQL基础](03-database/01-sql-fundamentals/) | DB introduction, SQLite setup, basic query/filter/sort, aggregate functions<br>数据库简介、SQLite环境配置、基础查询/过滤/排序、聚合函数 | 00:04 – 01:06:22 |
| [02 - Database Design 数据库设计](03-database/02-database-design/) | DISTINCT keyword, data redundancy, ER diagram, one-to-many & many-to-many, primary & foreign keys<br>去重关键字、数据冗余、ER实体关系图、一对多/多对多、主键外键 | 01:17:08 – 01:49:13 |
| [03 - Advanced Queries 高级查询](03-database/03-advanced-queries/) | Subqueries, IN operator, JOIN (INNER/LEFT/RIGHT/FULL), set operations<br>子查询、IN操作符、各类连接、集合运算 | 01:49:13 – 02:37:37 |
| [04 - Schema & Data Manipulation 表结构与数据操作](03-database/04-schema-and-data-manipulation/) | Database normalization, table creation, data types, constraints, ALTER / INSERT / DELETE / UPDATE<br>三大范式、建表语法、数据类型、字段约束、增删改改表语句 | 02:52:56 – 05:25:20 |
| [05 - Advanced Database Features 数据库高级特性](03-database/05-advanced-features/) | Triggers, soft delete, many-to-many implementation, VIEW, CTE, data anonymization<br>触发器、软删除、多对多实现、视图、公用表表达式、数据脱敏 | 05:49:09 – 07:31:17 |
| [06 - Performance & Transactions 性能优化与事务](03-database/06-performance-and-transactions/) | Index, VACUUM, ACID principle, concurrent isolation levels<br>索引、空间回收、事务ACID四大特性、并发隔离级别 | 07:31:17 – 09:01:45 |
| [07 - DB Systems & Security 主流数据库与安全防护](03-database/07-database-systems-and-security/) | MySQL vs PostgreSQL vs SQLite, stored procedures, replication & sharding, SQL injection prevention<br>三大数据库对比、存储过程、分库分表主从复制、SQL注入防御 | 09:01:45 – End |

---

## 🎯 Learning Objectives | 学习目标
### 目标
1. 系统性完整掌握计算机科学全套核心底层基础知识
2. 搭建完备前置知识体系，适配人工智能/机器学习研究生课程学习
3. 采用「课堂笔记 + 课后习题 + 小型实战项目」三位一体模式巩固知识点
4. 产出结构清晰、可复用、可传播的标准化自学资料，帮助同路线自学爱好者

### Objectives
1. Systematically master the full set of core underlying knowledge of computer science
2. Build a complete pre-knowledge system to support postgraduate AI & Machine Learning courses
3. Consolidate knowledge via a three-in-one learning mode: markdown notes + exercises + mini practical projects
4. Produce well-structured, reusable and shareable learning materials for fellow self-learners

---

## 🛠 Repository Structure & Usage | 仓库目录结构与使用方式
### Directory Standard for Each Subject Folder
每一个课程主题文件夹统一遵循以下目录规范
```bash
# Standard subfolders inside every single subject directory
- notes/       # Detailed learning notes (Markdown format) 课堂完整笔记（Markdown）
- exercises/   # After-class exercises & answer solutions 课后习题 + 参考答案
- projects/    # Mini practical projects to apply knowledge 小型实战落地项目
