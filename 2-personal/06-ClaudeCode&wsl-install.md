# 配置windows terminal代理（基于powershell 5.1版本）（*建议跳过，看下文升级powershell 7*）
## PowerShell 临时配置（Windows Terminal 通用）
1. 打开 Windows Terminal/PowerShell，逐行输入以下命令，每一行都带详细解释：
```powershell
# 1. 设置HTTP代理：给当前终端会话定义http代理地址，指向本地Clash的7897端口
$env:http_proxy="http://127.0.0.1:7897"
# 2. 设置HTTPS代理：加密的HTTPS请求也走同一个代理端口（Clash的代理地址统一用http://开头）
$env:https_proxy="http://127.0.0.1:7897"
# 3. 设置不走代理的地址：关键！避免本地、内网地址走代理导致访问失败
$env:no_proxy="localhost,127.0.0.1,::1,localaddress,.localdomain.com"
# 4. 同步大写变量：部分开发工具（Python/Node.js/Git）只认大写的环境变量名
$env:HTTP_PROXY=$env:http_proxy
$env:HTTPS_PROXY=$env:https_proxy
$env:NO_PROXY=$env:no_proxy
```
2. 配置完成后，用以下命令验证，返回 200 OK 就说明代理完全成功：
```powershell
# 核心验证：请求Google，返回HTTP/2 200就是成功
# 注意：PowerShell里必须用curl.exe（自带的curl是别名，会报错），CMD里直接用curl即可
curl.exe -I https://www.google.com
# 补充验证：查看当前出口IP，确认是不是你的代理节点IP（不是上海本地运营商IP）
curl.exe cip.cc
```
3. 如果不想用代理了，不用关终端，直接执行命令清空变量即可：
```powershell
Remove-Item Env:http_proxy,Env:https_proxy,Env:no_proxy,Env:HTTP_PROXY,Env:HTTPS_PROXY,Env:NO_PROXY
```

## 永久生效配置（每次打开终端自动加载）
测试临时配置成功后，再配置永久生效，不用每次打开终端都输一遍命令，和 WSL 里改.bashrc是一个逻辑。
原理是修改 PowerShell 的启动配置文件Profile，每次打开终端都会自动执行里面的代理配置，步骤如下：
1. 打开 PowerShell，先检查是否已有配置文件：
```powershell
Test-Path $PROFILE
```
- 返回True：已有配置文件，直接跳第 3 步
- 返回False：没有配置文件，执行第 2 步新建
2. 新建配置文件（仅上一步返回 False 时执行）：
```powershell
New-Item -Path $PROFILE -ItemType File -Force
```
3. 用记事本打开配置文件（小白友好，不用 vim）：
```powershell
notepad $PROFILE
```
在打开的记事本里，粘贴下面的代码，保存文件后关闭记事本：
```powershell
# --- Windows PowerShell 全局代理配置 ---
# 代理地址：端口改成你Clash Verge的实际端口
$proxy_addr = "http://127.0.0.1:7897"
# 设置代理环境变量
$env:http_proxy = $proxy_addr
$env:https_proxy = $proxy_addr
$env:no_proxy = "localhost,127.0.0.1,::1,localaddress,.localdomain.com"
# 同步大写变量，兼容所有开发工具
$env:HTTP_PROXY = $env:http_proxy
$env:HTTPS_PROXY = $env:https_proxy
$env:NO_PROXY = $env:no_proxy
# 启动时提示，确认配置加载成功
Write-Host "✅ 代理已自动加载: $proxy_addr"
```
5. 让配置立即生效（不用重启终端）：
```powershell
. $PROFILE
```
注意：开头有一个英文句号 + 空格，再跟$PROFILE
6. 最终验证：关闭所有 PowerShell/Windows Terminal 窗口，重新打开，会看到✅ 代理已自动加载的提示，再用curl.exe -I https://www.google.com测试即可。

**常见坑：PowerShell 提示「无法加载文件，执行策略禁止」**
- *本人未遇到*
这是 Windows 默认的脚本执行限制，解决方法：
1. 用管理员权限打开 PowerShell
2. 输入以下命令，按提示输入Y回车：
```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```
3. 这个配置是安全的，仅允许本地脚本运行，不会有系统风险。

- *本人遇到*
启动时输出的中文乱码
解决方式：
```powershell
notepad $PROFILE
```
在文件开头复制下列代码
```bash
# 强制所有PowerShell命令默认使用UTF-8编码（核心修复）
$PSDefaultParameterValues['*:Encoding'] = 'utf8'

# ====================== 全局UTF-8编码配置 ======================
# 切换终端代码页为UTF-8，隐藏多余输出
chcp 65001 | Out-Null
# 强制控制台输入/输出编码为UTF-8
[Console]::InputEncoding = [System.Text.Encoding]::UTF8
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
# 强制管道/外部工具输出编码为UTF-8
$OutputEncoding = [System.Text.Encoding]::UTF8
# 
```
另存为，选择格式为UFT-8 with BOM，让配置立即生效（不用重启终端）：
```powershell
. $PROFILE
```

# 安装WSL以及Ubuntu
1. 按 Win + S 搜索「启用或关闭 Windows 功能」
2. 找到「适用于 Linux 的 Windows 子系统」选项
3. 勾选后点击「确定」
4. 等待安装完成，按提示重启电脑
5. 安装 WSL（该步骤我未成功，可能通过windows更新自动带过了，可直接进入步骤6）
⚠️ 注意：请使用 PowerShell 或 Windows Terminal，不要使用 CMD。
打开 PowerShell（以管理员身份），执行：
`wsl --install`
安装完成后系统会提示重启，按要求重启电脑。
6. 安装Ubuntu发行版
重启后，打开 PowerShell 或 Windows Terminal：
`wsl --install -d Ubuntu`
安装完成后会要求创建 Linux 用户名和密码。
密码输入提示：输入密码时屏幕不会显示任何字符，这是 Linux 的安全特性，正常输入后按回车即可。
7. 进入Ubuntu
方式一：在 Windows Terminal 标签栏右侧的下拉菜单中选择「Ubuntu」
方式二：在 PowerShell 中直接输入：
`wsl`

# 配置wsl网络代理（可选，建议能配的配上）（*该步骤在本司环境下存在问题，建议跳过*）
## 步骤 1：先测试 WSL 和 Windows 的网络连通性
```bash
# 1. 从WSL的自动生成的DNS文件中提取Windows主机的IP（WSL2通过这个IP访问Windows）
# cat /etc/resolv.conf：读取DNS配置文件
# grep nameserver：过滤出包含nameserver的行（这行就是Windows的IP）
# awk '{print $2}'：提取该行的第2列（即IP地址）
WIN_HOST=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
# 2. 打印提取到的IP，确认是否成功（应该输出类似172.x.x.x的地址）
echo "Windows主机IP是: $WIN_HOST"
# 3. 测试WSL能不能ping通Windows（按Ctrl+C停止ping）
# 如果ping不通，说明Windows防火墙拦截了，暂时关闭Windows防火墙再试
ping $WIN_HOST
```
**注意，特殊限制下，可直接输入指定的windows IP**
`WIN_HOST=192.168.13.XX`

## 步骤 2：临时设置代理（当前终端生效，适合测试）
确认能 ping 通后，在当前终端执行
```bash
# 1. 再次确保WIN_HOST变量已定义（防止刚才没执行）
WIN_HOST=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
# 2. 设置HTTP代理
export http_proxy="http://$WIN_HOST:7897"
# 3. 设置HTTPS代理（注意：虽然是HTTPS，但Clash Verge的代理地址通常也是http://开头）
export https_proxy="http://$WIN_HOST:7897"
# 4. 设置NO_PROXY（关键！避免内网、本地地址走代理）
export no_proxy="localhost,127.0.0.1,::1,localaddress,.localdomain.com,$WIN_HOST"
# 5. 验证代理是否生效（如果返回Google的HTML代码，说明成功）
curl -I www.google.com
```

## 步骤 3：永久设置代理（每次打开终端自动生效）
```bash
vim ~/.bashrc
```
- 进入 Vim 后，按 G 键（大写，即 Shift+g），光标会跳到文件末尾；
- 按 o 键（小写），会在当前行下方插入新行，并进入「插入模式」（左下角会显示 -- INSERT --）；
- 复制下面的代码，右键粘贴到终端里：
```bash
# --- WSL代理配置（自动获取Windows主机IP）---
# 提取Windows主机的IP地址
export WIN_HOST=your ip
# 设置HTTP/HTTPS代理（7897改成你Clash Verge的实际端口）
export http_proxy="http://$WIN_HOST:7897"
export https_proxy="http://$WIN_HOST:7897"
# 设置不走代理的地址
export no_proxy="localhost,127.0.0.1,::1,localaddress,.localdomain.com,$WIN_HOST"
# 同时设置大写的变量名（有些软件只认大写）
export HTTP_PROXY=$http_proxy
export HTTPS_PROXY=$https_proxy
export NO_PROXY=$no_proxy
```
- 让配置立即生效：
```bash
# 重新加载 ~/.bashrc 文件，不用重启终端
source ~/.bashrc

# 再次验证
curl -I www.google.com
```


# 安装claude code(wsl)（*该步骤在本司环境下存在问题，建议跳过*）
**（wsl中对网络环境要求较复杂，本司环境存在问题，在最后Claude code无法连接到智谱的api进行回答，因此未继续研究，若有兴趣，可参考下列步骤）**
1. 更新包管理器和必要工具
```bash
sudo apt update && sudo apt install -y git curl jq gh nodejs
```
2. 安装Claude code
```bash
# 尝试下载脚本到本地
curl -fsSL https://claude.ai/install.sh -o claude-install.sh
# 查看文件头 10 行，确认是不是脚本
head claude-install.sh
# 添加执行权限
chmod +x claude-install.sh
# 执行安装
./claude-install.sh
```
提示：安装脚本会自动处理依赖，无需手动安装 Node.js。安装过程可能需要几分钟，请耐心等待。
**请确保代理处理Claude可用地区，例如美国**

3. 遇到的问题
```bash
✔ Claude Code successfully installed!

  Version: 2.1.81

  Location: ~/.local/bin/claude


  Next: Run claude --help to get started

⚠ Setup notes:
  • Native installation exists but ~/.local/bin is not in your PATH. Run:

  echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```
输入claude时报错，原因很清楚，**没加入path**
终端输入
```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

1. 安装cc switch
Github项目地址
https://github.com/farion1231/cc-switch/blob/main/docs/release-notes/v3.12.2-zh.md

- 下载linux版本安装包到本地
例如在D:\5-download\CC-Switch-v3.12.2-Linux-x86_64.deb
- 进入wsl将该安装包拷贝到wsl的文件系统中
例如`cp CC-Switch-v3.12.2-Linux-x86_64.deb ~/`
执行命令安装
```bash
sudo dpkg -i CC-Switch-v3.12.2-Linux-x86_64.deb
```
如果安装过程中提示 “依赖未满足”，执行以下命令自动修复依赖：
```bash
sudo apt -f install
```

**注意**
CC Switch 是 GTK GUI 应用，必须依赖 WSLg 提供的 X Server 才能渲染界面，DISPLAY 变量是空的，就等于 “没有显示器”，GUI 程序根本无法启动。

- 步骤 1：先确认 WSL 版本是 WSL2（必须前提）
打开 Windows 的PowerShell（管理员），执行：
```powershell
wsl --list --verbose
```
找到你的 Ubuntu 发行版，确认 VERSION 列是 2
若是1，看步骤2，若是2，看步骤3

- 步骤2：下载并安装 WSL2 Linux 内核更新包（微软官方）
打开微软官方下载链接（直接复制到浏览器打开）：
https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
下载完成后，双击运行 wsl_update_x64.msi，按提示完成安装（全程点 “下一步” 即可，无需修改默认选项）。
安装完成后，重启一次电脑（确保内核更新生效）
打开 Windows PowerShell（管理员权限），执行：
```powershell
# 先确保WSL已停止
wsl --shutdown
# 重新转换Ubuntu到WSL2（耐心等待，根据系统性能可能需要1-5分钟）
wsl --set-version Ubuntu 2
# 更新 WSL 到最新版本
wsl --update
```
✅ 成功的提示：
转换完成。（如果看到这个，说明 WSL2 升级成功）

- 步骤3：重新配置GUI环境
升级完成后，重新打开 WSL 的 Ubuntu 终端：
配置 DISPLAY 环境变量：
```bash
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0.0
export WAYLAND_DISPLAY=wayland-0
```
验证 GUI 环境：
在 WSL 终端执行：
```bash
echo $DISPLAY
echo $WAYLAND_DISPLAY
```
✅ 正常输出应该是：
DISPLAY：类似 :0 或 172.17.0.1:0.0
WAYLAND_DISPLAY：wayland-0
❌ 如果还是空，说明 WSLg 未启用，需要重新排查
执行下列命令
```bash
# 弹出眼睛窗口则正常
xeyes  
# 启动 CC Switch：
cc-switch
```

https://open.bigmodel.cn/api/anthropic
https://open.bigmodel.cn
8a92ba23b58e4a3cb1d7dcf2fc3b2634.N26pAXhcPxU8hUrh


sudo apt install -y libgtk-3-0 libwebkit2gtk-4.1-0 libayatana-appindicator3-1 libssl-dev libgdk-pixbuf2.0-0

我的windows terminal里面，有些中文字符无法正常显示。同时在WSL里面，也存在无法正常显示部分中文字符的问题。帮我看一下

curl https://open.bigmodel.cn/api/anthropic/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: 8a92ba23b58e4a3cb1d7dcf2fc3b2634.N26pAXhcPxU8hUrh" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "glm-4.7",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "介绍一下你自己"}]
  }'

172.26.160.1


WIN_PROXY_IP=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')


irm https://claude.ai/install.ps1 | iex


npm install -g @anthropic-ai/claude-code






