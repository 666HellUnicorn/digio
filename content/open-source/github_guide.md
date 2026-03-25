+++
date = '2026-02-09T10:00:00+08:00'
draft = false
title = 'GitHub 使用指南'
+++
# GitHub 使用指南

GitHub 是全球最大的代码托管平台，也是开源协作的核心工具。本指南将从零开始，帮助你掌握 GitHub 的核心功能和最佳实践。

---

## 1. GitHub 基础概念

### 1.1 什么是 GitHub？

GitHub 是基于 Git 的代码托管平台，提供：
- **代码存储**：安全地存储和管理代码
- **版本控制**：追踪代码变更历史
- **协作开发**：多人协作开发项目
- **开源社区**：参与和贡献开源项目

### 1.2 核心术语

| 术语 | 说明 |
|------|------|
| **Repository (仓库)** | 存放项目代码的地方，类似项目文件夹 |
| **Commit (提交)** | 保存代码变更的快照 |
| **Branch (分支)** | 独立的开发线，用于并行开发 |
| **Pull Request (PR)** | 请求将代码合并到主分支 |
| **Fork** | 复制他人的仓库到自己的账户 |
| **Clone** | 将远程仓库复制到本地 |
| **Push** | 将本地代码推送到远程仓库 |
| **Pull** | 从远程仓库拉取最新代码 |

---

## 2. 账户设置与初始化

### 2.1 创建 GitHub 账户

1. 访问 [GitHub.com](https://github.com)
2. 点击 "Sign up" 注册账户
3. 填写用户名、邮箱和密码
4. 验证邮箱地址

### 2.2 配置 SSH 密钥（推荐）

SSH 密钥让你无需每次输入密码即可推送代码。

**生成 SSH 密钥：**

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

**添加密钥到 GitHub：**

1. 复制公钥内容：
```bash
cat ~/.ssh/id_ed25519.pub
```

2. 在 GitHub 上添加：
   - 进入 Settings → SSH and GPG keys
   - 点击 "New SSH key"
   - 粘贴公钥内容并保存

**测试连接：**

```bash
ssh -T git@github.com
```

### 2.3 配置 Git 用户信息

```bash
git config --global user.name "你的用户名"
git config --global user.email "your_email@example.com"
```

---

## 3. 创建和管理仓库

### 3.1 创建新仓库

**通过网页创建：**

1. 点击右上角 "+" → "New repository"
2. 填写仓库名称（如 `my-awesome-project`）
3. 选择公开或私有
4. 可选：添加 README、.gitignore、License
5. 点击 "Create repository"

**通过命令行创建：**

```bash
# 初始化本地仓库
git init
git add .
git commit -m "Initial commit"

# 添加远程仓库
git remote add origin git@github.com:username/repo-name.git
git branch -M main
git push -u origin main
```

### 3.2 克隆现有仓库

```bash
# HTTPS 方式
git clone https://github.com/username/repo-name.git

# SSH 方式（推荐）
git clone git@github.com:username/repo-name.git
```

### 3.3 仓库设置

- **仓库描述**：简要说明项目用途
- **仓库标签**：添加主题标签（如 `python`, `machine-learning`）
- **默认分支**：设置主分支名称（推荐 `main`）
- **Wiki**：启用项目文档
- **Issues**：启用问题跟踪
- **Projects**：启用项目管理看板

---

## 4. Git 基础操作

### 4.1 工作流程

```text
工作区 → 暂存区 → 本地仓库 → 远程仓库
```

### 4.2 常用命令

**查看状态：**

```bash
git status
```

**添加文件到暂存区：**

```bash
# 添加所有文件
git add .

# 添加特定文件
git add filename.py

# 添加所有 .py 文件
git add *.py
```

**提交更改：**

```bash
git commit -m "描述你的更改"
```

**推送到远程：**

```bash
# 推送到当前分支
git push

# 推送到指定分支
git push origin main

# 首次推送并设置上游
git push -u origin main
```

**拉取最新代码：**

```bash
git pull
```

### 4.3 查看历史

```bash
# 查看提交历史
git log

# 查看简洁历史
git log --oneline

# 查看图形化历史
git log --graph --oneline --all

# 查看文件变更
git diff
```

---

## 5. 分支管理

### 5.1 创建和切换分支

```bash
# 创建新分支
git branch feature-branch

# 切换到分支
git checkout feature-branch

# 创建并切换（推荐）
git checkout -b feature-branch

# 切换到主分支
git checkout main
```

### 5.2 合并分支

```bash
# 切换到目标分支（如 main）
git checkout main

# 合并 feature-branch
git merge feature-branch

# 删除已合并的分支
git branch -d feature-branch
```

### 5.3 分支最佳实践

```text
main (主分支)
  └── develop (开发分支)
       ├── feature/user-auth (功能分支)
       ├── feature/payment (功能分支)
       └── hotfix/critical-bug (修复分支)
```

**分支命名规范：**

- `feature/功能描述`：新功能开发
- `bugfix/问题描述`：Bug 修复
- `hotfix/紧急问题`：紧急修复
- `refactor/重构内容`：代码重构
- `docs/文档内容`：文档更新

---

## 6. Pull Request (PR) 工作流

### 6.1 Fork 工作流（贡献开源项目）

1. **Fork 原仓库**：点击 Fork 按钮复制到你的账户
2. **克隆你的 Fork**：
```bash
git clone git@github.com:your-username/original-repo.git
cd original-repo
```

3. **添加上游仓库**：
```bash
git remote add upstream git@github.com:original-owner/original-repo.git
```

4. **创建功能分支**：
```bash
git checkout -b feature/my-contribution
```

5. **进行修改并提交**：
```bash
git add .
git commit -m "Add new feature"
git push origin feature/my-contribution
```

6. **创建 Pull Request**：
   - 在你的 Fork 页面点击 "Pull requests"
   - 点击 "New pull request"
   - 选择你的分支和目标分支
   - 填写 PR 标题和描述
   - 点击 "Create pull request"

7. **同步上游更新**：
```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### 6.2 PR 描述模板

```markdown
## 变更说明
简要描述这个 PR 的目的和内容。

## 变更类型
- [ ] 新功能
- [ ] Bug 修复
- [ ] 性能优化
- [ ] 文档更新
- [ ] 代码重构

## 测试情况
- [ ] 已在本地测试通过
- [ ] 已添加单元测试
- [ ] 已更新相关文档

## 相关 Issue
Closes #123

## 截图（如适用）
[上传截图]

## 检查清单
- [ ] 代码遵循项目规范
- [ ] 已通过所有测试
- [ ] 已更新文档
```

---

## 7. Issues 管理

### 7.1 创建 Issue

1. 进入仓库的 "Issues" 标签页
2. 点击 "New issue"
3. 填写标题和详细描述
4. 选择标签（如 `bug`, `enhancement`, `documentation`）
5. 分配给相关人员（可选）
6. 点击 "Submit new issue"

### 7.2 Issue 模板

**Bug 报告模板：**

```markdown
## Bug 描述
清晰简洁地描述 bug 是什么。

## 复现步骤
1. 进入 '...'
2. 点击 '....'
3. 滚动到 '....'
4. 看到错误

## 期望行为
清晰描述你期望发生什么。

## 实际行为
清晰描述实际发生了什么。

## 环境信息
- OS: [例如 Windows 11]
- 浏览器: [例如 Chrome 120]
- 版本: [例如 v1.2.3]

## 截图
如果适用，添加截图来帮助解释问题。

## 附加信息
添加任何其他关于问题的信息。
```

**功能请求模板：**

```markdown
## 功能描述
清晰简洁地描述你想要的功能。

## 问题背景
描述这个功能解决的问题。

## 期望解决方案
详细描述你希望如何实现这个功能。

## 替代方案
描述你考虑过的任何替代解决方案或功能。

## 附加信息
添加任何其他关于功能请求的信息。
```

### 7.3 Issue 标签管理

**常用标签：**

| 标签 | 用途 |
|------|------|
| `bug` | Bug 报告 |
| `enhancement` | 功能增强 |
| `documentation` | 文档相关 |
| `good first issue` | 适合新手 |
| `help wanted` | 需要帮助 |
| `priority: high` | 高优先级 |
| `wontfix` | 不会修复 |

---

## 8. GitHub Actions 自动化

### 8.1 创建工作流

在 `.github/workflows/` 目录下创建 YAML 文件：

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    
    - name: Run tests
      run: |
        pytest
```

### 8.2 常用 Actions

**自动部署：**

```yaml
- name: Deploy to GitHub Pages
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./dist
```

**发布到 PyPI：**

```yaml
- name: Publish to PyPI
  uses: pypa/gh-action-pypi-publish@release/v1
  with:
    password: ${{ secrets.PYPI_API_TOKEN }}
```

---

## 9. GitHub Pages 静态网站

### 9.1 启用 GitHub Pages

1. 进入仓库 Settings
2. 找到 "Pages" 部分
3. 选择 Source（Branch 或 Workflow）
4. 选择分支和目录（如 `main` 分支的 `/docs` 文件夹）
5. 点击 Save

### 9.2 部署流程

**使用 Actions 自动部署：**

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Build
      run: npm run build
    
    - name: Deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./dist
```

---

## 10. 最佳实践

### 10.1 Commit 消息规范

**Conventional Commits 格式：**

```
<type>(<scope>): <subject>

<body>

<footer>
```

**类型说明：**

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(auth): add OAuth login` |
| `fix` | Bug 修复 | `fix(api): resolve timeout issue` |
| `docs` | 文档更新 | `docs(readme): update installation guide` |
| `style` | 代码格式 | `style: format code with prettier` |
| `refactor` | 重构 | `refactor(user): simplify validation logic` |
| `test` | 测试 | `test(api): add unit tests for user module` |
| `chore` | 构建/工具 | `chore: update dependencies` |

### 10.2 .gitignore 最佳实践

**Python 项目：**

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
env/
ENV/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Project specific
config.local.yaml
secrets.yaml
```

**Node.js 项目：**

```gitignore
# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Build
dist/
build/
*.tgz

# Environment
.env
.env.local
.env.*.local

# IDE
.vscode/
.idea/
```

### 10.3 安全最佳实践

**不要提交敏感信息：**

- API 密钥
- 数据库密码
- 私钥文件
- 个人信息

**使用环境变量：**

```yaml
# .github/workflows/deploy.yml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

**使用 Secrets：**

1. Settings → Secrets and variables → Actions
2. 点击 "New repository secret"
3. 添加名称和值

### 10.4 README 编写规范

**标准 README 结构：**

```markdown
# 项目名称

简短的项目描述

## 功能特性
- 功能 1
- 功能 2
- 功能 3

## 安装

```bash
npm install my-project
```

## 使用

```javascript
const myProject = require('my-project');
myProject.doSomething();
```

## 配置

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `apiKey` | API 密钥 | `null` |
| `timeout` | 超时时间 | `5000` |

## 贡献指南

欢迎贡献！请查看 [CONTRIBUTING.md](CONTRIBUTING.md)

## 许可证

MIT License
```

---

## 11. 高级功能

### 11.1 GitHub Codespaces

云端开发环境，无需本地配置。

**使用步骤：**

1. 点击仓库的 "Code" 按钮
2. 选择 "Codespaces" 标签
3. 点击 "Create codespace on main"
4. 等待环境创建完成

### 11.2 GitHub Copilot

AI 编程助手，提供智能代码补全。

**启用步骤：**

1. 进入 GitHub Settings
2. 找到 "Copilot" 部分
3. 点击 "Start trial" 或订阅
4. 安装 VS Code 扩展

### 11.3 GitHub Projects

项目管理工具，支持看板和表格视图。

**创建项目：**

1. 进入仓库的 "Projects" 标签页
2. 点击 "New project"
3. 选择模板（Board 或 Table）
4. 添加列和任务

---

## 12. 故障排查

### 12.1 常见问题

**问题 1：推送时提示权限被拒绝**

```bash
# 解决方案：检查远程仓库 URL
git remote -v

# 如果是 HTTPS，改为 SSH
git remote set-url origin git@github.com:username/repo.git
```

**问题 2：合并冲突**

```bash
# 解决方案：手动解决冲突后
git add .
git commit -m "Resolve merge conflict"
git push
```

**问题 3：无法拉取最新代码**

```bash
# 解决方案：强制更新
git fetch --all
git reset --hard origin/main
```

### 12.2 调试技巧

**查看远程仓库信息：**

```bash
git remote -v
```

**查看分支跟踪关系：**

```bash
git branch -vv
```

**清理无用分支：**

```bash
git remote prune origin
```

---

## 13. 资源链接

### 官方文档
- [GitHub 官方文档](https://docs.github.com)
- [Git 官方文档](https://git-scm.com/doc)

### 学习资源
- [GitHub Skills](https://skills.github.com)
- [GitHub Learning Lab](https://lab.github.com)

### 工具推荐
- [GitHub Desktop](https://desktop.github.com) - 图形化 Git 客户端
- [GitKraken](https://www.gitkraken.com) - 可视化 Git 工具
- [SourceTree](https://www.sourcetreeapp.com) - 免费 Git 客户端

---

## 14. 总结

GitHub 是现代软件开发不可或缺的工具。掌握以下核心技能：

✅ **基础操作**：创建仓库、克隆、提交、推送
✅ **分支管理**：创建、切换、合并分支
✅ **协作开发**：Pull Request、Code Review
✅ **问题跟踪**：Issues、标签、里程碑
✅ **自动化**：GitHub Actions、CI/CD
✅ **最佳实践**：Commit 规范、安全、文档

持续实践和探索，你将更高效地使用 GitHub 进行开发和协作！

---

**最后更新：2026-02-09**
