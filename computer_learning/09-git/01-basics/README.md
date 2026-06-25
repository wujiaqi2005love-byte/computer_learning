# 01 - Git 基础

> 🏗️ 从零开始掌握 Git 的核心命令和概念

---

## 1. Git 是什么？

Git 是 **Linus Torvalds**（Linux 内核的创造者）在 2005 年开发的分布式版本控制系统。

**核心思想：Git 不存储文件差异，而是存储文件快照（Snapshot）。**

```
传统 VCS（如 SVN）：           Git：
v1 → diff → v2 → diff → v3     v1 的快照 | v2 的快照 | v3 的快照
（每个版本存变化量）             （每个版本存完整状态）
```

这意味着 Git 的几乎所有操作都在**本地**完成，速度极快。

---

## 2. Git 的三个状态

```
工作目录               暂存区                  Git 仓库
(Working Directory) → (Staging Area) →      (.git/)
                       git add               git commit

     ↖─────────────────── git checkout / restore ────────────↙
```

| 状态 | 含义 | 文件在哪里 |
|------|------|-----------|
| **Modified**（已修改） | 改过但没暂存 | 工作目录 |
| **Staged**（已暂存） | 标记为要提交 | 暂存区 |
| **Committed**（已提交） | 安全存入仓库 | `.git/` 目录 |

---

## 3. 基本命令

### 3.1 创建仓库

```bash
# 在当前目录初始化新仓库
git init

# 克隆已有仓库
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git
```

### 3.2 记录变更

```bash
# 查看文件状态
git status                     # 哪些文件改了、哪些暂存了
git status -s                  # 简短模式

# 暂存文件
git add <file>                 # 添加单个文件
git add .                      # 添加当前目录所有变更
git add -p                     # 交互式，逐个 hunk 选择是否暂存

# 提交
git commit -m "提交信息"        # 提交暂存区内容
git commit -am "提交信息"       # 跳过 git add，直接提交所有已跟踪文件的修改
                               # ⚠️ 新文件不会自动加入！
```

### 3.3 查看历史

```bash
git log                        # 完整历史
git log --oneline              # 一行一条，简洁
git log --oneline --graph      # 带分支图
git log --oneline -5           # 只看最近 5 条

git diff                       # 工作目录 vs 暂存区
git diff --staged              # 暂存区 vs 最新提交
git diff <commit1> <commit2>   # 两个提交之间的差异
```

### 3.4 撤销操作

```bash
# 取消暂存（从暂存区移回工作目录）
git restore --staged <file>    # 新命令（推荐）
git reset HEAD <file>          # 旧命令

# 丢弃工作目录修改（回到上次提交的状态）
git restore <file>             # 新命令
git checkout -- <file>         # 旧命令

# 修改最后一次提交（改信息或补充文件）
git commit --amend -m "新信息"
git commit --amend --no-edit   # 不改信息，只补充暂存的文件
```

---

## 4. `.gitignore` 文件

某些文件不应该被版本控制（编译产物、依赖包、密钥等）：

```bash
# 创建 .gitignore 文件
cat > .gitignore << 'EOF'
# 依赖
node_modules/
__pycache__/
*.pyc

# 编译产物
build/
dist/
*.class

# 系统文件
.DS_Store
Thumbs.db

# 环境变量
.env
*.secret

# IDE
.vscode/
.idea/
EOF
```

---

## 5. 分支（Branch）

分支是 Git 最强大的特性之一。它让你可以在不影响主线的情况下做实验。

```bash
# 查看分支
git branch               # 本地分支列表
git branch -a            # 包括远程分支

# 创建分支
git branch <name>        # 创建但不切换
git checkout -b <name>   # 创建并切换
git switch -c <name>     # 新命令（推荐）

# 切换分支
git checkout <name>
git switch <name>        # 新命令（推荐）

# 合并分支
git merge <name>         # 将 <name> 分支合并到当前分支

# 删除分支
git branch -d <name>     # 安全删除（已合并的才能删）
git branch -D <name>     # 强制删除
```

### 分支模型示意

```
main:    ●───●───●───●───●───●
                      ↘
feature:               ●───●───●
                              ↗
main:    ●───●───●───●───●───●───●  (merge 之后)
```

---

## 6. 合并与冲突

### 6.1 快进合并（Fast-forward）

如果 main 在 feature 创建后没有新提交，合并时 Git 直接把 main 指针移到 feature 的位置，无需创建新提交。

```bash
# 创建新分支，做个提交
git checkout -b feature
echo "new feature" >> file.txt
git add . && git commit -m "Add feature"

# 切回 main 合并
git checkout main
git merge feature     # Fast-forward 合并
```

### 6.2 三方合并（3-way merge）

如果两个分支都有各自的提交，Git 会创建一个**合并提交**。

```bash
# main 和 feature 都有新提交
git checkout main
git merge feature     # 自动合并 or 产生冲突
```

### 6.3 解决冲突

当两个分支修改了同一个文件的同一行，Git 无法自动判断该用哪个版本：

```
<<<<<<< HEAD
这是 main 分支的版本
=======
这是 feature 分支的版本
>>>>>>> feature
```

解决步骤：
1. 手动编辑文件，删除标记，保留正确的内容
2. `git add <file>` 标记为已解决
3. `git commit` 完成合并

---

## 7. Rebase vs Merge

```
Merge:                           Rebase:
main:  ●──●──●──●               main:  ●──●──●──●──●──●
            ↘   ↗                            （线性的）
feature:     ●──●
          创建合并提交                  把 feature 的提交接到 main 后面
```

| | Merge | Rebase |
|---|-------|--------|
| 历史 | 保留真实的分支历史 | 创建线性历史，更干净 |
| 安全性 | ✅ 安全，不改已有提交 | ⚠️ 改写历史，协作中慎用 |
| 何时用 | 公共分支、团队代码 | 个人分支、整理本地提交 |

```bash
git rebase main        # 把当前分支变基到 main 上
git rebase -i HEAD~3   # 交互式 rebase，整理最近 3 个提交
```

---

## 8. 标签（Tag）

标签用于标记重要的版本节点（如发布版本）。

```bash
git tag v1.0.0                   # 轻量标签
git tag -a v1.0.0 -m "发布说明"   # 附注标签（推荐）
git push origin v1.0.0           # 推送单个标签
git push origin --tags           # 推送所有标签
```

---

## 9. Stash（暂存工作现场）

当你正在改代码但突然需要切分支时，先把未提交的修改存起来：

```bash
git stash              # 暂存当前修改
git stash list         # 查看 stash 列表
git stash pop          # 恢复最近一次 stash 并删除
git stash apply        # 恢复但不删除
git stash drop         # 删除 stash
```

---

## 📝 常用命令速查

| 操作 | 命令 |
|------|------|
| 查看状态 | `git status` |
| 暂存所有 | `git add .` |
| 提交 | `git commit -m "..."` |
| 查看日志 | `git log --oneline` |
| 查看差异 | `git diff` |
| 创建并切换分支 | `git switch -c <name>` |
| 合并分支 | `git merge <name>` |
| 暂存工作 | `git stash` |
| 恢复文件 | `git restore <file>` |
| 取消暂存 | `git restore --staged <file>` |
