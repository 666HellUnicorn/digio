+++
date = '2026-04-15T23:27:13+08:00'
draft = false
title = 'PowerShell 终端美化完全指南：从安装到配置'
+++

# PowerShell 终端美化完全指南

将 Windows 原生终端打造为类 Linux（oh-my-zsh）的高效开发环境。

---

## 效果预览

美化后的 PowerShell 终端将具备以下特性：

- **彩色提示符**：显示当前目录、Git 分支、执行时间
- **智能补全**：输入时自动联想历史命令
- **图标支持**：文件、文件夹、Git 状态可视化图标
- **快速跳转**：智能目录记忆与模糊搜索

---

## 第一步：安装核心工具

### 1.1 安装 PowerShell 7

```powershell
# 使用 winget 安装最新版 PowerShell
winget install --id Microsoft.Powershell --source winget

# 安装完成后，在开始菜单搜索 "PowerShell" 启动
```

### 1.2 安装 Oh My Posh

```powershell
# 安装主题美化工具
winget install JanDeDobbeleer.OhMyPosh --source winget

# 安装 Nerd Font（解决图标显示问题）
oh-my-posh font install

# 推荐选择：JetBrainsMono Nerd Font 或 MesloLGS NF
```

### 1.3 安装辅助工具

```powershell
# 智能目录跳转
winget install ajeetdsouza.zoxide

# 模糊搜索工具
winget install junegunn.fzf

# Git 状态显示
Install-Module posh-git -Force
```

---

## 第二步：配置 PowerShell 配置文件

### 2.1 打开配置文件

```powershell
# 如果文件不存在，会自动创建
notepad $PROFILE
```

### 2.2 完整配置内容

将以下内容复制到 `$PROFILE` 文件中：

```powershell
# ============================================
# PowerShell 美化配置文件
# ============================================

# 1. 初始化 Oh My Posh（主题美化）
oh-my-posh init pwsh --config $env:POSH_THEMES_PATH\ys.omp.json | Invoke-Expression

# 2. 配置 PSReadLine（智能补全）
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle InlineView
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

# 3. 初始化 Zoxide（智能目录跳转）
Invoke-Expression (& { (zoxide init powershell | Out-String) })

# 4. 导入 Posh-Git（Git 状态显示）
Import-Module posh-git

# 5. 设置别名（可选）
Set-Alias -Name z -Value __zoxide_z
Set-Alias -Name zi -Value __zoxide_zi

# 6. 启用 FZF（模糊搜索）
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'
```

### 2.3 保存并生效

```powershell
# 保存文件后，重新加载配置
. $PROFILE

# 或者重启 PowerShell
```

---

## 第三步：配置 Windows Terminal

### 3.1 设置默认字体

1. 打开 **Windows Terminal**
2. 按 `Ctrl + ,` 打开设置
3. 选择 **PowerShell** 配置文件
4. 点击 **外观**
5. 将 **字体** 设置为 `JetBrainsMono Nerd Font`

### 3.2 设置默认启动

1. 在设置中选择 **启动**
2. 将 **默认配置文件** 设置为 **PowerShell**
3. 将 **默认终端应用程序** 设置为 **Windows Terminal**

---

## 第四步：验证安装

### 4.1 检查各组件状态

```powershell
# 检查 PowerShell 版本
$PSVersionTable.PSVersion

# 检查 Oh My Posh 版本
oh-my-posh version

# 检查 Zoxide 版本
zoxide --version

# 检查 FZF 版本
fzf --version
```

### 4.2 测试功能

```powershell
# 测试智能补全：输入 "cd Do" 应该显示历史目录

# 测试 Zoxide：输入 "z doc" 跳转到 Documents 目录

# 测试 FZF：按 Ctrl+R 搜索历史命令

# 测试 Git 显示：进入 Git 仓库，查看提示符是否显示分支名
```

---

## 第五步：常用快捷键

### 5.1 命令编辑

| 快捷键 | 功能 |
|--------|------|
| `Ctrl + A` | 跳到行首 |
| `Ctrl + E` | 跳到行尾 |
| `Ctrl + U` | 删除光标前所有字符 |
| `Ctrl + K` | 删除光标后所有字符 |
| `Ctrl + W` | 删除前一个单词 |
| `Ctrl + L` | 清屏 |

### 5.2 历史命令

| 快捷键 | 功能 |
|--------|------|
| `↑ / ↓` | 浏览历史命令 |
| `Ctrl + R` | 模糊搜索历史（FZF） |
| `Tab` | 自动补全 |

### 5.3 Windows Terminal

| 快捷键 | 功能 |
|--------|------|
| `Ctrl + Shift + T` | 新建标签页 |
| `Ctrl + Shift + W` | 关闭标签页 |
| `Ctrl + Tab` | 切换标签页 |
| `Shift + Alt + +` | 横向分屏 |
| `Shift + Alt + -` | 纵向分屏 |

---

## 常见问题解决

### 问题 1：执行策略限制

**错误信息**：
```
无法加载文件，因为在此系统上禁止运行脚本
```

**解决方法**：
```powershell
# 修改执行策略
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 确认选择 Y
```

### 问题 2：图标显示为方框

**原因**：未安装 Nerd Font 或字体设置错误

**解决方法**：
1. 确认已运行 `oh-my-posh font install`
2. 在 Windows Terminal 设置中，将字体改为 `JetBrainsMono Nerd Font`
3. 重启终端

### 问题 3：Oh My Posh 未生效

**检查步骤**：
```powershell
# 1. 检查配置文件是否存在
Test-Path $PROFILE

# 2. 查看配置文件内容
cat $PROFILE

# 3. 手动加载配置
. $PROFILE
```

### 问题 4：启动速度慢

**优化方法**：
```powershell
# 使用静态主题（更快）
oh-my-posh init pwsh --print | Invoke-Expression

# 或者禁用部分模块
# 注释掉 $PROFILE 中不需要的模块
```

---

## 进阶配置

### 自定义主题

```powershell
# 查看所有可用主题
Get-PoshThemes

# 使用其他主题（例如 powerlevel10k）
oh-my-posh init pwsh --config $env:POSH_THEMES_PATH\powerlevel10k.omp.json | Invoke-Expression
```

### 自定义提示符

创建自定义主题文件 `~/.mytheme.omp.json`：

```json
{
  "$schema": "https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/schema.json",
  "blocks": [
    {
      "type": "prompt",
      "alignment": "left",
      "segments": [
        {
          "type": "path",
          "style": "powerline",
          "powerline_symbol": "\uE0B0",
          "foreground": "#ffffff",
          "background": "#61AFEF",
          "template": " {{ .Path }} "
        }
      ]
    }
  ]
}
```

---

## 完整配置一键安装脚本

```powershell
# 保存为 Install-PowerShellTools.ps1 并执行

# 安装 PowerShell 7
winget install --id Microsoft.Powershell --source winget

# 安装 Oh My Posh
winget install JanDeDobbeleer.OhMyPosh --source winget

# 安装字体
oh-my-posh font install

# 安装 Zoxide
winget install ajeetdsouza.zoxide

# 安装 FZF
winget install junegunn.fzf

# 安装 Posh-Git
Install-Module posh-git -Force

# 创建配置文件
$profileContent = @'
# Oh My Posh
oh-my-posh init pwsh --config $env:POSH_THEMES_PATH\ys.omp.json | Invoke-Expression

# PSReadLine
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle InlineView

# Zoxide
Invoke-Expression (& { (zoxide init powershell | Out-String) })

# Posh-Git
Import-Module posh-git

# 别名
Set-Alias -Name z -Value __zoxide_z
Set-Alias -Name zi -Value __zoxide_zi
'@

# 确保目录存在
if (!(Test-Path (Split-Path $PROFILE))) {
    New-Item -ItemType Directory -Path (Split-Path $PROFILE) -Force
}

# 写入配置文件
$profileContent | Out-File -FilePath $PROFILE -Encoding utf8

# 设置执行策略
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force

Write-Host "安装完成！请重启 PowerShell 并运行 . `$PROFILE" -ForegroundColor Green
```

---

## 参考资料

- [Oh My Posh 官方文档](https://ohmyposh.dev/)
- [PowerShell 文档](https://docs.microsoft.com/powershell/)
- [Windows Terminal 文档](https://docs.microsoft.com/windows/terminal/)
- [Zoxide GitHub](https://github.com/ajeetdsouza/zoxide)
- [FZF GitHub](https://github.com/junegunn/fzf)

---

> **提示**：配置完成后，建议重启 Windows Terminal 以确保所有更改生效。
