# 09 - Git（版本控制）

> 📖 自学 Git 的笔记与实践记录
> 🛠 工具：Git CLI、GitHub
> 📅 开始日期：2025.06

---

## 什么是 Git？

**Git** 是一个**分布式版本控制系统**（Distributed Version Control System, DVCS）。简单来说，它记录文件的每一次修改历史，让你可以：

- 📝 **追踪变更**：谁在什么时候改了什么
- ⏪ **回退历史**：回到任意一个历史版本
- 🌿 **分支开发**：在不影响主线的情况下实验新功能
- 🤝 **多人协作**：多人同时改代码，智能合并冲突
- ☁️ **远程备份**：推送到 GitHub/GitLab 等远程仓库

---

## 核心概念速览

```
┌─────────────┐      git push       ┌─────────────────┐
│  本地仓库    │  ───────────────→   │   远程仓库       │
│  (Local)    │  ←───────────────   │   (Remote)       │
│             │      git pull       │  GitHub/GitLab   │
│  Working    │                     │                  │
│  Directory  │                     │                  │
│  + Staging  │                     │                  │
│  + Commits  │                     │                  │
└─────────────┘                     └─────────────────┘
```

| 概念 | 说明 |
|------|------|
| **Working Directory** | 你正在编辑的文件，工作区 |
| **Staging Area** | 暂存区，`git add` 后的文件放在这里，准备提交 |
| **Commit** | 一次快照，记录某个时间点的文件状态 |
| **Branch** | 分支，一条独立的开发线 |
| **Remote** | 远程仓库，通常是 GitHub/GitLab 上的副本 |
| **Clone** | 把远程仓库完整复制到本地 |
| **Push/Pull** | 推送（上传）/ 拉取（下载）本地与远程之间的更改 |

---

## 课程章节导航

| # | 章节 | 重点 |
|---|------|------|
| 1 | [Git 基础](01-basics/) | 安装配置、init/clone、add/commit、log/diff、分支基础 |
| 2 | [远程与协作](02-remote-and-collaboration/) | remote、push/pull、SSH 密钥、force push、常见场景 |

---

## 📂 目录结构

```
09-git/
├── README.md                      ← 你在这里
├── 01-basics/                     ← Git 基础命令与概念
├── 02-remote-and-collaboration/   ← 远程操作与多人协作
└── exercises/                     ← 练习记录
```

---

## 🎯 学习目标

- 理解 Git 的核心概念（快照、分支、合并）
- 熟练掌握日常 Git 操作
- 学会与 GitHub 远程仓库交互
- 能够处理常见的协作场景（冲突解决、回退、force push）

---

## 🛠 环境准备

```bash
# macOS 自带 git，检查版本
git --version

# 配置用户名和邮箱（提交时会附上这些信息）
git config --global user.name "Jiaqi Wu"
git config --global user.email "your-email@example.com"
```
