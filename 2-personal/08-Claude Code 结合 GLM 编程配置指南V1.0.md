# Claude Code 结合 GLM 编程配置指南


## 一、 Claude Code CLI 工具的下载与安装

Claude Code 是一款命令行界面（CLI）工具，专为 AI 辅助编程设计。它能够直接在终端中运行，理解您的代码库，并协助完成代码编写、重构和调试等任务。

### 1. 安装前提条件

在开始安装之前，请确保您的系统满足以下基本要求：

• Node.js 环境：必须安装Node环境v18以上版本


### 2. 安装Claude Code CLI方法一

推荐通过 npm 进行全局安装。为了提高在国内的下载速度，建议先安装 `nrm` 工具并切换到淘宝源或其他国内镜像源。

**步骤 1：安装 nrm 并切换镜像源**
在终端中执行以下命令安装 nrm：
```bash
npm install -g nrm
```
安装完成后，使用 nrm 切换到淘宝源（taobao）：
```bash
nrm use taobao
```

**步骤 2：安装Claude Code**
配置好国内源后，执行以下命令全局安装 Claude Code：
```bash
npm install -g @anthropic-ai/claude-code
```
安装完成后，您就可以在终端中直接使用 `claude` 命令。


### 3. 安装Claude Code CLI方法二

官方推荐使用原生安装脚本，该方法支持在后台自动更新，确保您始终使用最新版本的 Claude Code。

**对于 macOS、Linux 用户：**
在终端中执行以下命令进行安装：
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**对于 Windows 用户：**
Windows 用户在安装前需要确保已安装 Git for Windows。
如果您使用 PowerShell，请执行：
```powershell
irm https://claude.ai/install.ps1 | iex
```
如果您使用命令提示符（CMD），请执行：
```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### 4. 验证及启用

安装完成后，您可以在终端中输入 `claude --version` 若显示版本号则表示安装成功。
在终端中输入 `claude` 命令启动Claude Code CLI工具

## 二、 订阅智谱 GLM 进行 AI Coding

由于 Claude 官方服务在国内使用可能存在网络和支付等限制，许多开发者选择使用国内优秀的平替大模型。智谱 AI 推出的 GLM Coding Plan 是专为 AI 编程打造的订阅套餐，完美兼容 Claude Code 等工具。


### 1. 订阅流程

订阅 GLM Coding Plan 的流程非常简便：
1. 访问智谱大模型开放平台（https://www.bigmodel.cn/glm-coding?ic=1B6TG6IPIW），点击右上角进行注册或登录。
2. 登录后，前往"Coding Plan"套餐详情页，选择适合您的套餐并完成支付。 体验卡(https://www.bigmodel.cn/activity/trial-card/O2261ENZVS)
3. 进入个人中心页面，导航至"API Keys"选项卡，点击"创建新 API Key"并妥善保存生成的密钥，这将在后续配置中使用。

###

## 三、 通过 CC Switch 配置 GLM 模型

虽然可以通过修改配置文件或运行脚本来让 Claude Code 接入 GLM 模型，但这种方式较为繁琐。**CC Switch** 是一款开源的跨平台桌面应用，专门用于管理 Claude Code、Codex、Gemini CLI 等工具的配置。它提供了直观的可视化界面，让您能够一键切换不同的模型供应商，彻底告别手动编辑配置文件的烦恼。

### 1. CC Switch 的下载与安装

CC Switch 基于 Tauri 2 构建，支持 Windows、macOS 和 Linux 操作系统。

**Windows 安装：**
请访问 CC Switch 的 GitHub Releases 页面（https://github.com/farion1231/cc-switch/releases），下载最新版本的 `.msi` 安装包或 `.zip` 绿色免安装版。

**macOS 安装：**
推荐使用 Homebrew 进行安装，只需在终端执行以下命令：
```bash
brew tap farion1231/ccswitch
brew install --cask cc-switch
```
您也可以在 Releases 页面手动下载 `.dmg` 文件进行安装 [3]。

**Linux 安装：**
Arch Linux 用户可以通过 `paru -S cc-switch-bin` 安装。其他发行版用户可以在 Releases 页面下载对应的 `.deb`、`.rpm`、`.AppImage` 或 `.flatpak` 文件。

### 2. 在 CC Switch 中配置智谱 GLM

安装并打开 CC Switch 后，请按照以下步骤配置智谱 GLM 模型：

1. **添加供应商**：在 CC Switch 主界面，点击右上角的 "+" 按钮以添加新的模型供应商。
2. **选择预设**：在预设供应商列表中，找到并选择 "Zhipu GLM"。
3. **填写 API Key**：在 "API Key" 输入框中，填入您之前在智谱开放平台获取的 API 密钥。
4. **确认请求地址**：系统会自动填充请求地址为 `https://open.bigmodel.cn/api/anthropic`，请确保该地址正确且不以斜杠结尾。
5. **配置模型映射**：在模型设置区域，将"主模型"、"Haiku 默认模型"、"Sonnet 默认模型"和"Opus 默认模型"全部填写为 `glm-5.1`（或您订阅支持的其他 GLM 模型版本）。由于 GLM API 兼容了 Claude 的接口格式，这样设置可以确保 Claude Code 的各种请求都能正确路由到 GLM 模型。
6. **保存配置**：点击右下角的 "+ 添加" 按钮保存配置。

### 3. 启用与测试

配置完成后，您将在 CC Switch 的主界面看到刚刚添加的 "Zhipu GLM" 供应商。
1. 点击该供应商卡片上的 **"启用"** 按钮，将其设置为当前使用的模型。
2. 启用成功后，您可以直接点击卡片上的终端图标（`>_`）打开命令行。
3. 在终端中输入 `claude` 启动 Claude Code。
4. 为了验证配置是否成功，您可以在 Claude Code 的提示符下输入："当前模型是什么？"。如果它回答"当前模型是 glm-5.1"，则说明配置已完美生效，您可以开始使用 GLM 模型进行 AI 编程了。

---
