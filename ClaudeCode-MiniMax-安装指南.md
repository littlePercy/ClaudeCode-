# Claude Code + MiniMax API 安装配置指南

> Windows 环境完整教程 · 0 基础可懂 · 含全部踩坑记录

---

## 目录

1. [前置条件](#1-前置条件)
2. [安装 Node.js](#2-安装-nodejs)
3. [安装 Claude Code](#3-安装-claude-code)
4. [安装 Git（提供 Git Bash）](#4-安装-git提供-git-bash)
5. [配置环境变量（注册表）](#5-配置环境变量注册表)
6. [创建启动包装脚本](#6-创建启动包装脚本)
7. [配置 PowerShell Profile](#7-配置-powershell-profile)
8. [配置 settings.json](#8-配置-settingsjson)
9. [配置代理（关键！）](#9-配置代理关键)
10. [创建桌面快捷启动器](#10-创建桌面快捷启动器)
11. [验证安装](#11-验证安装)
12. [常见问题排查](#12-常见问题排查)
13. [附录：踩坑记录](#13-附录踩坑记录)

---

## 1. 前置条件

| 项目 | 说明 | 状态 |
|------|------|------|
| Windows 10/11 电脑 | 64 位系统 | **必须** |
| MiniMax API Key | 从 MiniMax 开放平台获取，格式 `sk-api-xxxxx` | **必须** |
| 代理软件（Clash/V2Ray 等） | 用于让 Claude Code 完成初始认证检查，需监听 `127.0.0.1:7890` | **必须** |
| 管理员权限 | 用于修改系统环境变量 | **必须** |

> **为什么需要代理？**
> Claude Code 交互模式启动时会先连接 `api.anthropic.com`（Anthropic 官方服务器）做认证检查。这个检查**不走**你配置的 `ANTHROPIC_BASE_URL`。在中国大陆无法直连 `api.anthropic.com`，必须通过代理。**但实际的 API 调用会走 MiniMax**，所以代理只用于认证检查这一步。

---

## 2. 安装 Node.js

Claude Code 是一个 Node.js 程序，需要先安装 Node.js 运行时。

### 方法 A：官网下载（推荐新手）

1. 访问 https://nodejs.org
2. 下载 **LTS 版本**（长期支持版）
3. 双击安装，一路「下一步」即可
4. 安装时确保勾选 **"Add to PATH"** 选项

### 方法 B：命令行安装

```bash
winget install OpenJS.NodeJS.LTS
```

### 验证安装

打开**新的** CMD 或 PowerShell 窗口：

```bash
node --version
# 应输出类似: v22.22.2
```

> ⚠️ 安装 Node.js 后，**必须关闭并重新打开**终端窗口，新的 PATH 才会生效。

---

## 3. 安装 Claude Code

打开 CMD 或 PowerShell，执行：

```bash
npm install -g @anthropic-ai/claude-code
```

安装完成后验证：

```bash
claude --version
# 应输出类似: 2.1.195
```

> ⚠️ 如果提示 `'npm' 不是内部或外部命令`，说明 Node.js 没有正确添加到 PATH。请重新安装 Node.js 并确保勾选 "Add to PATH"，然后**重新打开终端**。

安装后 claude.exe 的常见路径：
- 标准 npm 安装：`C:\Users\{用户名}\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code\bin\claude.exe`
- WorkBuddy 管理：`C:\Users\{用户名}\.workbuddy\binaries\node\versions\{版本号}\node_modules\@anthropic-ai\claude-code\bin\claude.exe`

---

## 4. 安装 Git（提供 Git Bash）

Claude Code 在 Windows 上需要 Git Bash 来执行内部 shell 命令。

### 安装 Git

1. 访问 https://git-scm.com/download/win
2. 下载 64-bit Git for Windows
3. 双击安装，大部分选项保持默认即可

### 找到 bash.exe 的路径

常见路径：

```
# 默认安装路径
C:\Program Files\Git\bin\bash.exe

# 如果装在其他位置（例如 D 盘）
D:\Tools\Git\bin\bash.exe
```

> 🔴 **踩坑 #1：Git 安装在非默认路径**
> 如果 Git 没有装在 `C:\Program Files\Git`，Claude Code 会找不到 bash.exe。必须通过环境变量 `CLAUDE_CODE_GIT_BASH_PATH` 手动指定路径。

验证：

```bash
where bash
# 或
dir "C:\Program Files\Git\bin\bash.exe"
```

---

## 5. 配置环境变量（注册表）

这是最核心的一步。在 Windows 注册表中设置**用户级**环境变量。

### 方法 A：图形界面设置（推荐新手）

1. 按 `Win + R`，输入 `sysdm.cpl`，回车
2. 点击「高级」选项卡 →「环境变量」
3. 在「用户变量」区域，点击「新建」，逐个添加：

| 变量名 | 变量值 | 说明 |
|--------|--------|------|
| `ANTHROPIC_BASE_URL` | `https://api.minimaxi.com/anthropic` | MiniMax 的 Anthropic 兼容端点 |
| `ANTHROPIC_API_KEY` | `sk-api-你的key` | 你的 MiniMax API Key |
| `ANTHROPIC_MODEL` | `MiniMax-M3` | 使用的模型 |
| `ANTHROPIC_SMALL_FAST_MODEL` | `MiniMax-M3` | 快速模型 |
| `CLAUDE_CODE_GIT_BASH_PATH` | `C:\Program Files\Git\bin\bash.exe` | Git Bash 路径（改成你的实际路径！） |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | `1` | 启用实验性 Agent Teams |
| `HTTP_PROXY` | `http://127.0.0.1:7890` | 代理地址 |
| `HTTPS_PROXY` | `http://127.0.0.1:7890` | 代理地址 |
| `NO_PROXY` | `api.minimaxi.com,minimaxi.com,localhost,127.0.0.1` | MiniMax 直连不走代理 |

### 方法 B：PowerShell 命令设置（快捷）

```powershell
# 替换 API_KEY 和 GIT_BASH_PATH 为你的实际值
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://api.minimaxi.com/anthropic", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "sk-api-你的key", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_MODEL", "MiniMax-M3", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_SMALL_FAST_MODEL", "MiniMax-M3", "User")
[System.Environment]::SetEnvironmentVariable("CLAUDE_CODE_GIT_BASH_PATH", "C:\Program Files\Git\bin\bash.exe", "User")
[System.Environment]::SetEnvironmentVariable("CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS", "1", "User")
[System.Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://127.0.0.1:7890", "User")
[System.Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://127.0.0.1:7890", "User")
[System.Environment]::SetEnvironmentVariable("NO_PROXY", "api.minimaxi.com,minimaxi.com,localhost,127.0.0.1", "User")
```

> 🔴 **踩坑 #2：路径用反斜杠，不要用正斜杠！**
> Windows 路径必须用 `\`，例如 `C:\Program Files\Git\bin\bash.exe`。如果用 `/` 或 `//`，Claude Code 会找不到文件。

> 🔴 **踩坑 #3：settings.json 的 env 块不能控制 Claude Code 自身！**
> `settings.json` 的 `env` 字段只影响**子进程**，不影响 Claude Code 自己的 API 连接。必须用系统环境变量。

> ⚠️ 设置完后**已打开的终端不会自动刷新**，必须关闭重开。

---

## 6. 创建启动包装脚本

由于环境变量可能被各种来源覆盖，需要创建**包装脚本**强制设置正确变量。

### 6.1 创建存放目录

```bash
mkdir C:\Users\你的用户名\bin
```

### 6.2 创建 claude.cmd（CMD 包装器）

在 `C:\Users\你的用户名\bin\` 下创建 `claude.cmd`：

```cmd
@ECHO off
REM Claude Code - MiniMax API 启动包装器

REM 设置代理（认证检查需要）
SET HTTP_PROXY=http://127.0.0.1:7890
SET HTTPS_PROXY=http://127.0.0.1:7890
SET NO_PROXY=api.minimaxi.com,minimaxi.com,localhost,127.0.0.1

REM 设置 MiniMax API 配置
SET ANTHROPIC_BASE_URL=https://api.minimaxi.com/anthropic
SET ANTHROPIC_API_KEY=sk-api-你的key
SET ANTHROPIC_MODEL=MiniMax-M3
SET ANTHROPIC_SMALL_FAST_MODEL=MiniMax-M3
SET CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
SET CLAUDE_CODE_GIT_BASH_PATH=C:\Program Files\Git\bin\bash.exe

REM 启动 claude.exe（替换为你的实际路径）
"C:\Program Files\nodejs\node_modules\@anthropic-ai\claude-code\bin\claude.exe" %*
```

### 6.3 创建 claude.ps1（PowerShell 包装器）

在同一个目录下创建 `claude.ps1`：

```powershell
# Claude Code - MiniMax API 启动包装器 (PowerShell)

# 设置代理
$env:HTTP_PROXY  = 'http://127.0.0.1:7890'
$env:HTTPS_PROXY = 'http://127.0.0.1:7890'
$env:NO_PROXY    = 'api.minimaxi.com,minimaxi.com,localhost,127.0.0.1'

# 设置 MiniMax API 配置
$env:ANTHROPIC_BASE_URL            = 'https://api.minimaxi.com/anthropic'
$env:ANTHROPIC_API_KEY              = 'sk-api-你的key'
$env:ANTHROPIC_MODEL                = 'MiniMax-M3'
$env:ANTHROPIC_SMALL_FAST_MODEL     = 'MiniMax-M3'
$env:CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS = '1'
$env:CLAUDE_CODE_GIT_BASH_PATH      = 'C:\Program Files\Git\bin\bash.exe'

# 启动
& "C:\Program Files\nodejs\node_modules\@anthropic-ai\claude-code\bin\claude.exe" @args
exit $LASTEXITCODE
```

### 6.4 把 bin 目录加入 PATH

在环境变量设置中，编辑用户变量 `Path`，在最前面添加：

```
C:\Users\你的用户名\bin
```

> 🔴 **踩坑 #4：PATH 顺序问题**
> npm 安装后会在 npm 目录下也生成 `claude.cmd` 和 `claude.ps1`。如果 npm 目录排在你的 `bin` 目录前面，系统会优先找到 npm 的脚本。**解决方案：**把 `bin` 目录放在 PATH 最前面，或同时修改 npm 目录下的脚本。

> 🔴 **踩坑 #5：PATH 格式损坏**
> 用 Git Bash 编辑 PATH 时可能把 `\` 变成 `/`。**解决方案：**用 Windows 图形界面或 PowerShell 设置 PATH。

---

## 7. 配置 PowerShell Profile

### 7.1 检查执行策略

```powershell
# 以管理员身份打开 PowerShell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

> 🔴 **踩坑 #6：PowerShell 执行策略限制**
> 默认策略 `Restricted` 会阻止执行任何 .ps1 脚本，包括 Profile。必须改为 `RemoteSigned`。

### 7.2 创建/编辑 Profile

文件路径：`C:\Users\你的用户名\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`

```powershell
# Claude Code - MiniMax API 配置
# 每次 PowerShell 启动时自动设置

# 添加 claude 包装器目录到 PATH
$claudeDir = 'C:\Users\你的用户名\bin'
if ($env:Path -notlike "*$claudeDir*") {
    $env:Path = $claudeDir + ';' + $env:Path
}

# 设置代理
$env:HTTP_PROXY  = 'http://127.0.0.1:7890'
$env:HTTPS_PROXY = 'http://127.0.0.1:7890'
$env:NO_PROXY    = 'api.minimaxi.com,minimaxi.com,localhost,127.0.0.1'

# 设置 MiniMax API 配置
$env:ANTHROPIC_BASE_URL            = 'https://api.minimaxi.com/anthropic'
$env:ANTHROPIC_API_KEY              = 'sk-api-你的key'
$env:ANTHROPIC_MODEL                = 'MiniMax-M3'
$env:ANTHROPIC_SMALL_FAST_MODEL     = 'MiniMax-M3'
$env:CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS = '1'
$env:CLAUDE_CODE_GIT_BASH_PATH      = 'C:\Program Files\Git\bin\bash.exe'
```

---

## 8. 配置 settings.json

文件路径：`C:\Users\你的用户名\.claude\settings.json`

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "env": {
    "ANTHROPIC_API_KEY": "sk-api-你的key",
    "ANTHROPIC_BASE_URL": "https://api.minimaxi.com/anthropic",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "MiniMax-M3",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "MiniMax-M3",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "MiniMax-M3",
    "ANTHROPIC_MODEL": "MiniMax-M3",
    "ANTHROPIC_SMALL_FAST_MODEL": "MiniMax-M3",
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe",
    "HTTP_PROXY": "http://127.0.0.1:7890",
    "HTTPS_PROXY": "http://127.0.0.1:7890",
    "NO_PROXY": "api.minimaxi.com,minimaxi.com,localhost,127.0.0.1"
  }
}
```

> ⚠️ JSON 文件中的 Windows 路径必须用**双反斜杠** `\\`。

---

## 9. 配置代理（关键！）

这是整个安装过程中**最容易踩坑**的一步。

### 为什么需要代理？

| 模式 | 启动方式 | 是否连接 api.anthropic.com | 需要代理 |
|------|----------|--------------------------|----------|
| 非交互模式 | `claude -p "问题"` | ❌ 不需要 | 不需要 |
| 交互模式 | `claude` | ✅ 需要（认证检查） | **需要** |

> 🔴 **踩坑 #7（最大的坑）：交互模式需要代理连接 api.anthropic.com**
>
> Claude Code 交互模式启动时，会先连接 `api.anthropic.com` 做认证检查。**这个连接不走 `ANTHROPIC_BASE_URL`**。
>
> 在中国大陆无法直连，如果不设代理，会报：
> ```
> Failed to connect to api.anthropic.com: ERR_BAD_REQUEST
> ```
>
> **解决方案：**设置 `HTTP_PROXY` 和 `HTTPS_PROXY` 环境变量指向代理软件。

### 代理配置要求

1. 安装并启动代理软件（Clash / V2Ray / 其他）
2. 确保代理监听在 `127.0.0.1:7890`（Clash 默认端口）
3. 代理需要有**美国节点**（Anthropic 限制地区访问）
4. 设置环境变量（已在步骤 5/6/7 中完成）

### NO_PROXY 的作用

设置 `NO_PROXY=api.minimaxi.com` 后，发往 MiniMax 的 API 请求会**直连**，不走代理（MiniMax 在国内可以直连，走代理反而更慢）。

### 验证代理

```bash
# 验证代理是否运行
curl http://127.0.0.1:7890

# 验证能否通过代理访问 api.anthropic.com
curl --proxy http://127.0.0.1:7890 https://api.anthropic.com/v1/messages
# 返回 401 是正常的（说明能连上，只是 key 不对）
```

> 🔴 **踩坑 #8：代理软件没运行时设置了代理变量**
> 如果代理变量指向 `127.0.0.1:7890` 但代理软件没运行，所有网络请求都会失败。

> 🔴 **踩坑 #9：之前"清除代理"的方案是错的**
> 早期排查时认为代理变量会干扰 MiniMax 连接，于是全部清除。结果交互模式的认证检查无法到达 api.anthropic.com。**正确做法：**设置代理变量 + 用 NO_PROXY 排除 MiniMax 域名。

---

## 10. 创建桌面快捷启动器

在桌面创建 `ClaudeCode-MiniMax.bat`：

```cmd
@ECHO off
TITLE Claude Code - MiniMax-M3

REM 设置代理
SET HTTP_PROXY=http://127.0.0.1:7890
SET HTTPS_PROXY=http://127.0.0.1:7890
SET NO_PROXY=api.minimaxi.com,minimaxi.com,localhost,127.0.0.1

REM 设置 MiniMax API 配置
SET ANTHROPIC_BASE_URL=https://api.minimaxi.com/anthropic
SET ANTHROPIC_API_KEY=sk-api-你的key
SET ANTHROPIC_MODEL=MiniMax-M3
SET ANTHROPIC_SMALL_FAST_MODEL=MiniMax-M3
SET CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
SET CLAUDE_CODE_GIT_BASH_PATH=C:\Program Files\Git\bin\bash.exe

echo ==========================================
echo   Claude Code - MiniMax-M3
echo   BASE_URL: %ANTHROPIC_BASE_URL%
echo   PROXY:    http://127.0.0.1:7890
echo ==========================================
echo.

REM 启动（替换为你的实际 claude.exe 路径）
"C:\Program Files\nodejs\node_modules\@anthropic-ai\claude-code\bin\claude.exe" %*
```

---

## 11. 验证安装

### 验证清单

1. **关闭所有终端窗口**（重要！）
2. 确保**代理软件已启动**并选择了美国节点
3. 打开**新的** PowerShell 窗口
4. 逐项验证：

```powershell
# 1. 检查 Claude Code 版本
claude --version
# 期望: 2.1.195 或更高

# 2. 检查环境变量
$env:ANTHROPIC_BASE_URL
# 期望: https://api.minimaxi.com/anthropic

$env:ANTHROPIC_API_KEY
# 期望: sk-api-...（你的 key）

$env:CLAUDE_CODE_GIT_BASH_PATH
# 期望: C:\Program Files\Git\bin\bash.exe

$env:HTTP_PROXY
# 期望: http://127.0.0.1:7890

# 3. 非交互模式测试（不需要代理）
claude -p "说你好"
# 期望: 你好！

# 4. 交互模式测试（需要代理）
claude
# 期望: 进入交互界面，没有报错
```

> 💡 如果步骤 3 通过但步骤 4 失败，检查代理是否正常运行。

---

## 12. 常见问题排查

### 问题 1：`'claude' 不是内部或外部命令`

**原因：** claude.exe 所在目录不在 PATH 中。

**解决：**
1. 找到 claude.cmd 位置：`where.exe claude`
2. 把对应目录加入 PATH
3. 重新打开终端

### 问题 2：`Failed to connect to api.anthropic.com`

**原因：** 交互模式的认证检查无法到达 api.anthropic.com。

**解决：**
1. 确认代理软件正在运行：`curl http://127.0.0.1:7890`
2. 确认环境变量已设置：`$env:HTTPS_PROXY` 应为 `http://127.0.0.1:7890`
3. 确认代理选择了美国节点
4. 确认 `NO_PROXY` 包含 `api.minimaxi.com`

### 问题 3：Git Bash 相关错误

**原因：** `CLAUDE_CODE_GIT_BASH_PATH` 未设置或路径错误。

**解决：**
1. 找到 bash.exe：`where.exe bash`
2. 设置 `CLAUDE_CODE_GIT_BASH_PATH`
3. 路径用反斜杠 `\`

### 问题 4：PowerShell 无法执行脚本

**原因：** 执行策略为 Restricted。

**解决：**
```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 问题 5：修改了环境变量但不生效

**原因：** 已打开的终端不会自动刷新。

**解决：** 关闭所有终端，重新打开。或重启电脑。

### 问题 6：非交互模式正常但交互模式失败

**原因：** 交互模式需要代理连接 api.anthropic.com 做认证检查。

**解决：** 确保 `HTTP_PROXY` 和 `HTTPS_PROXY` 已设置，且代理正在运行。

### 问题 7：代理端口不是 7890

| 代理软件 | 默认 HTTP 端口 |
|----------|---------------|
| Clash for Windows | 7890 |
| V2RayN | 10809 |
| Shadowsocks | 1080 |

把所有配置中的 `7890` 替换为你的实际端口。

---

## 13. 附录：踩坑记录

| # | 问题 | 症状 | 根因 | 解决方案 |
|---|------|------|------|----------|
| 1 | claude 命令找不到 | `CommandNotFoundException` | claude.exe 目录不在 PATH | 添加到 PATH + 重开终端 |
| 2 | PowerShell 无法执行脚本 | 执行策略错误 | 默认策略 Restricted | `Set-ExecutionPolicy RemoteSigned` |
| 3 | settings.json 无效 | 仍连接 api.anthropic.com | env 块只影响子进程 | 改用系统环境变量 |
| 4 | 注册表变量不生效 | 新变量在已打开的终端中不存在 | 终端不自动刷新 | 关闭重开 + 创建 Profile |
| 5 | PATH 顺序错误 | 找到 npm 的脚本而非包装器 | npm 目录排在前面 | bin 目录放 PATH 最前面 |
| 6 | PATH 格式损坏 | 部分程序找不到 | Git Bash 把 `\` 改成 `/` | 用 PowerShell 重写 PATH |
| 7 | 代理变量干扰 | 连不上任何 API | 代理没运行但变量已设 | 启动代理或清除变量 |
| 8 | Git Bash 路径找不到 | 各种 shell 错误 | Git 装在非默认路径 | 手动设置 `CLAUDE_CODE_GIT_BASH_PATH` |
| 9 | 路径双斜杠 | bash.exe 找不到 | 注册表值为 `D://app...` | 用反斜杠重写 |
| 10 | 交互模式认证失败 | `Failed to connect to api.anthropic.com` | 交互模式需连接官方做认证，之前清除了代理 | **设置**代理变量 + NO_PROXY 排除 MiniMax |
| 11 | 非交互正常交互失败 | `claude -p` 正常，`claude` 失败 | 非交互模式跳过认证检查 | 同上，必须设置代理 |

### 核心经验

问题拆成三层：
1. **环境变量是否正确**——注册表 + 包装器 + Profile 三重保障
2. **Git Bash 路径是否正确**——`CLAUDE_CODE_GIT_BASH_PATH` 必须指向真实存在的 bash.exe
3. **代理是否配置正确**——交互模式需要代理到达 api.anthropic.com，但 MiniMax API 调用直连（通过 NO_PROXY 排除）

---

*MiniMax Anthropic 兼容端点: `https://api.minimaxi.com/anthropic` · 模型: `MiniMax-M3`*
