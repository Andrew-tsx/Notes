# Claude Code 安装步骤

## 安装WSL以及Ubuntu

1. 按 Win + S 搜索「启用或关闭 Windows 功能」
2. 找到「适用于 Linux 的 Windows 子系统」选项、虚拟机平台
3. 勾选后点击「确定」
4. 等待安装完成，按提示重启电脑
    老版本的windows可执行下列两条命令开启相关功能

    ```powershell
    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    ```

5. 安装 WSL（该步骤我未成功，可能通过windows更新自动带过了，可直接进入步骤6）
⚠️ 注意：请使用 PowerShell 或 Windows Terminal，不要使用 CMD。
打开 PowerShell（以管理员身份），执行：
`wsl --install`
安装完成后系统会提示重启，按要求重启电脑。
6. 安装Ubuntu发行版
   重启后，打开 PowerShell 或 Windows Terminal：
   - 先确定一下是否能用wsl命令，查看已发行的版本
   `wsl --list --online`
   - 将wsl版本设定为wsl2
   `wsl --set-default-version 2`
   *若失败了可直接去官网下载升级包*
   打开微软官方下载链接（直接复制到浏览器打开）：
   <https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi>
   - 安装Ubuntu
    `wsl --install -d Ubuntu`
    没反应的话多等一会儿，可能需要时间
    安装完成后会要求创建 Linux 用户名和密码。
    密码输入提示：输入密码时屏幕不会显示任何字符，这是 Linux 的安全特性，正常输入后按回车即可。
7. 进入Ubuntu
    方式一：在 Windows Terminal 标签栏右侧的下拉菜单中选择「Ubuntu」
    方式二：在 PowerShell 中直接输入：
    `wsl`
8. 查看安装的Linux子系统版本详情
    `wsl --list --verbose`

## 安装docker

1. docker-desktop下载地址
    <https://www.docker.com/products/docker-desktop/>
    选择windows版本进行下载
    安装时保持默认就行，不要选allow windows containers to be used with this installation
2. 打开docker desktop
    会提示wsl版本过旧需要升级，按照提示打开terminal运行命令
    `wsl --update`
3. 注册账号，修改镜像地址（**可选**）
    用魔法打开注册页面，按照要求注册即可
    登录软件后建议更改镜像安装地址，否则默认在C盘
    settings-resources-advanced
4. 验证是否安装成功
    打开terminal，输入`docker --version`查看

## 安装Node.js

### 在docker里面装Node.js

1. 打开terminal，拉取镜像
    `docker pull node:24-alpine`
2. 创建一个Node.js容器
    `docker run -it --rm --entrypoint sh node:24-alpine`
3. 验证node和npm版本

```bash
    # Verify the Node.js version:
    node -v # Should print "v24.14.1".
    # Verify npm version:
    npm -v # Should print "11.11.0".
```

### 直接下安装包装在windows环境中

官网地址
<https://nodejs.org/en/download/>
验证方式同上

## 安装Claude Code

进入命令行界面，安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

运行如下命令，查看安装结果，若显示版本号则表示安装成功
`claude --version`

## 安装cc switch

资源链接
<https://github.com/farion1231/cc-switch/releases/download/v3.10.3/CC-Switch-v3.10.3-Windows.msi>

根据需要选择要接入的模型和平台，输入相应的api key即可


