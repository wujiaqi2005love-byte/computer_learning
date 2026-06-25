# 02 - 远程仓库与协作

> 🔑 核心技能：将本地代码推送到 GitHub、从 GitHub 拉取、配置 SSH、处理常见远程操作场景

---

## 1. 远程仓库的概念

远程仓库（Remote Repository）是托管在网络上的 Git 仓库副本。最流行的托管平台是 **GitHub**。

```
你的电脑（本地）                         GitHub（远程）
┌──────────────────┐                   ┌──────────────────┐
│  main 分支       │  ── git push ──→  │  origin/main     │
│  new_start 分支  │                   │                  │
│  commit eb2f5dc  │  ←─ git pull ──  │  commit eb2f5dc  │
└──────────────────┘                   └──────────────────┘
```

---

## 2. 查看和管理远程仓库

```bash
# 查看远程仓库列表
git remote -v
# 输出示例：
# origin  https://github.com/wujiaqi2005love-byte/computer_learning.git (fetch)
# origin  https://github.com/wujiaqi2005love-byte/computer_learning.git (push)

# 添加远程仓库
git remote add origin <url>

# 修改远程仓库 URL
git remote set-url origin <new-url>

# 查看远程仓库上有哪些分支
git ls-remote origin
```

`origin` 是约定的默认远程仓库名，你可以有多个远程仓库（如 `origin`、`upstream`、`backup`）。

---

## 3. 推送与拉取

### 3.1 基本推送

```bash
# 推送当前分支到远程
git push

# 推送指定分支到远程的指定分支
git push origin <local-branch>:<remote-branch>
git push origin main:main
```

### 3.2 拉取远程更新

```bash
# 拉取并合并
git pull

# 先拉取（不合并），再手动检查
git fetch origin
git diff origin/main   # 查看远程和本地的差异
git merge origin/main  # 手动合并
```

---

## 4. 🔑 SSH 密钥配置（实操记录）

HTTPS 方式推送时，每次都需要输入密码（或 token）。更推荐使用 **SSH 方式**，配置一次，永久免密。

### 4.1 为什么需要 SSH？

- **HTTPS**：通过 443 端口，走 HTTP 协议，可能受代理影响
- **SSH**：通过 22 端口，加密通信，配置密钥后可免密推送

### 4.2 实操步骤

```bash
# Step 1: 生成 SSH 密钥
ssh-keygen -t ed25519 -C "your-github-email@example.com" -f ~/.ssh/id_ed25519 -N ""

# Step 2: 查看公钥（复制到 GitHub）
cat ~/.ssh/id_ed25519.pub
# 输出示例：
# ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... your-email@example.com

# Step 3: 测试连接
ssh -T git@github.com
# 成功输出：Hi username! You've successfully authenticated...
```

### 4.3 将公钥添加到 GitHub

1. 打开 https://github.com/settings/keys
2. 点击 **"New SSH key"**
3. 粘贴公钥内容，取个名字（如 "MacBook"）
4. 点击 **"Add SSH key"**

### 4.4 切换远程 URL 为 SSH

```bash
# 从 HTTPS 切换到 SSH
git remote set-url origin git@github.com:username/repo.git

# URL 格式对照：
# HTTPS: https://github.com/username/repo.git
# SSH:   git@github.com:username/repo.git
```

---

## 5. 💥 Force Push（强制推送）— 今天实操

### 5.1 什么是 Force Push？

`git push --force` 会**用本地分支的内容完全覆盖远程分支**，无论远程上有什么。

⚠️ **危险操作！** 远程分支上别人的提交会被永久删除。

### 5.2 使用场景

| 场景 | 是否适合 force push |
|------|-------------------|
| 自己的个人仓库，想重置远程内容 | ✅ 可以 |
| 修改了本地 commit 历史（rebase 后） | ✅ 需要 |
| 团队协作仓库，main 分支 | ❌ 绝对禁止 |
| 自己的 feature 分支 | ⚠️ 谨慎，通知队友 |

### 5.3 实操案例：清空云端仓库，推送本地内容

```bash
# 目标：清除远程 main 分支的所有内容，用本地 new_start 分支替代

# Step 1: 检查当前状态
git remote -v                          # 确认远程地址
git branch                             # 确认当前分支为 new_start

# Step 2: 配置 SSH 密钥（见上一节）

# Step 3: 切换远程 URL 为 SSH
git remote set-url origin git@github.com:wujiaqi2005love-byte/computer_learning.git

# Step 4: 强制推送（用本地 new_start 覆盖远程 main）
git push origin new_start:main --force

# 命令解读：
#   origin          → 远程仓库名
#   new_start:main  → 将本地 new_start 分支 推送到 远程 main 分支
#   --force         → 强制覆盖，不检查冲突

# Step 5: 验证
git ls-remote origin
# 输出：
# eb2f5dc...  HEAD
# eb2f5dc...  refs/heads/main
```

### 5.4 Force Push 的安全替代方案

```bash
# 更安全：使用 --force-with-lease
# 只有当远程分支和你上次 fetch 的状态一致时才执行
# 防止不小心覆盖别人的提交
git push --force-with-lease origin new_start:main

# 如果 --force-with-lease 被拒绝，说明有人在你之后推送了代码
# 先 git fetch 查看，再决定是否继续
```

---

## 6. 代理环境下 Git 的配置（问题排查记录）

### 6.1 问题背景

在公司/学校网络或使用代理工具（Clash、V2Ray 等）时，HTTPS 方式的 Git 推送可能失败。

### 6.2 遇到的错误

```bash
# 错误 1：直接连接超时（无代理时无法访问 GitHub）
fatal: unable to access 'https://github.com/.../': 
  Failed to connect to github.com port 443: Couldn't connect to server

# 错误 2：通过 HTTP 代理后 TLS 握手失败
fatal: unable to access 'https://github.com/.../': 
  LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443
```

### 6.3 排查过程

```bash
# 1. 检查系统代理设置
networksetup -getwebproxy Wi-Fi
# 发现：127.0.0.1:7890（FlClash）

# 2. 检查代理进程
ps aux | grep clash
# 确认 FlClash 正在运行

# 3. HTTTPS + 代理 走不通
#    CONNECT tunnel 建立成功
#    但 TLS 握手阶段失败（LibreSSL SSL_ERROR_SYSCALL）
#    怀疑 FlClash 的 TLS 实现与 git 自带的 LibreSSL 不兼容

# 4. 检查是否能直连
curl --connect-timeout 10 -I https://github.com
# 超时 — 网络环境无法直连 GitHub

# 5. 尝试 SSH 方式
ssh -T git@github.com
# 直连成功！SSH 不走 443 端口，可以直连 GitHub
```

### 6.4 解决方案：使用 SSH 替代 HTTPS

```bash
# 为 Git 配置 HTTP/HTTPS 代理（如果以后某些仓库需要走代理）
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 但对于 GitHub，切换到 SSH 方式避免 TLS 问题
git remote set-url origin git@github.com:username/repo.git
```

### 6.5 如果 SSH 也需要走代理

```bash
# 在 ~/.ssh/config 中配置
Host github.com
  HostName github.com
  User git
  ProxyCommand nc -X connect -x 127.0.0.1:7890 %h %p
```

---

## 7. 常用命令速查

```bash
# 远程信息
git remote -v                          # 查看远程仓库
git remote add <name> <url>            # 添加远程
git remote set-url <name> <url>        # 修改 URL
git remote remove <name>               # 删除远程

# 推送
git push                               # 推送当前分支
git push origin <branch>               # 推送到指定远程分支
git push origin --delete <branch>      # 删除远程分支

# 拉取
git pull                               # 拉取并合并
git fetch origin                       # 只拉取不合并
git pull --rebase                      # 拉取并 rebase（推荐）

# 查看远程信息
git ls-remote origin                   # 查看远程所有分支和 HEAD
git remote show origin                 # 查看远程详细信息

# 配置
git config --global user.name "..."    # 设置用户名
git config --global user.email "..."   # 设置邮箱
git config --global http.proxy "..."   # 设置 HTTP 代理
git config --global --list             # 查看所有全局配置
```

---

## 📝 本次实操总结

| 操作 | 命令 |
|------|------|
| 生成 SSH 密钥 | `ssh-keygen -t ed25519 -C "..."` |
| 添加公钥到 GitHub | GitHub Settings → SSH Keys |
| 切换远程 URL | `git remote set-url origin git@github.com:...` |
| 强制推送本地覆盖远程 | `git push origin new_start:main --force` |
| 验证结果 | `git ls-remote origin` |

**关键教训：**
- HTTPS + 代理 + git 自带的 LibreSSL 可能出现 TLS 兼容问题
- SSH 直连往往是更可靠的选择
- Force push 是危险操作，只在个人仓库使用
- 配置 SSH 密钥后可以免密推送，比 HTTPS 更安全方便
