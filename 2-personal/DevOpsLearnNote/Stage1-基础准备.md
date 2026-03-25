# 阶段一：基础准备 - 详细学习计划

> 学习周期：10天（2周）
> 每日学习时间：5小时
> 开始日期：____年__月__日
> 完成日期：____年__月__日

---

## 学习目标

完成本阶段学习后，你将能够：
- 熟练使用 Linux 常用命令和系统管理
- 理解 Linux 文件系统和权限管理
- 掌握网络基础知识和协议原理
- 能够编写基础 Shell 脚本
- 理解 HTTP 协议和负载均衡原理

---

## 第1天：Linux 环境准备与文件操作

### 学习目标
- [ ] 搭建学习环境（虚拟机/云服务器）
- [ ] 掌握文件和目录操作命令
- [ ] 理解 Linux 目录结构

### 知识点详解

#### 1. Linux 目录结构
```
/                 # 根目录
├── bin/          # 常用命令（ls, cp, cat等）
├── etc/          # 系统配置文件
├── home/         # 普通用户家目录
├── root/         # root 用户家目录
├── var/          # 变量文件（日志、缓存）
├── usr/          # 用户程序
├── tmp/          # 临时文件
└── opt/          # 可选软件包
```

#### 2. 常用命令

| 命令          | 功能                  | 常用参数                             |
| ------------- | --------------------- | ------------------------------------ |
| `ls`          | 列出文件              | `-la`（详细+隐藏）、`-h`（人类可读） |
| `cd`          | 切换目录              | `-`（返回上一次）、`~`（家目录）     |
| `pwd`         | 显示当前目录          |                                      |
| `mkdir`       | 创建目录              | `-p`（递归创建）                     |
| `rmdir`       | 删除空目录            |                                      |
| `rm`          | 删除文件/目录         | `-r`（递归）、`-f`（强制）           |
| `cp`          | 复制文件/目录         | `-r`（递归）、`-a`（保留属性）       |
| `mv`          | 移动/重命名           |                                      |
| `touch`       | 创建空文件/更新时间戳 |                                      |
| `cat`         | 查看文件内容          |                                      |
| `more`/`less` | 分页查看文件          |                                      |
| `head`/`tail` | 查看文件开头/结尾     | `-n` 指定行数                        |

### 练习任务（5小时）

| 任务           | 预计时间 | 说明                                                              |
| -------------- | -------- | ----------------------------------------------------------------- |
| 环境搭建       | 1小时    | 安装 VirtualBox/VMware，创建 CentOS 7/Ubuntu 20.04 虚拟机（2C4G） |
| 文件操作练习   | 1.5小时  | 在家目录下创建目录树结构，练习各种文件命令                        |
| 目录结构探索   | 1小时    | 使用 ls / 浏览各个目录，理解用途                                  |
| 命令熟练度练习 | 1小时    | 反复练习常用命令，形成肌肉记忆                                    |
| 总结记录       | 0.5小时  | 记录学习笔记和遇到的问题                                          |

### 练习场景

```bash
# 场景1：创建项目目录结构
cd ~
mkdir -p ~/projects/{dev,test,prod}
mkdir -p ~/projects/dev/{backend,frontend,docs}

# 场景2：文件操作练习
cd ~/projects/dev
touch app.py config.yaml README.md
ls -la

# 场景3：文件复制和移动
cp app.py ../test/
mv README.md docs/

# 场景4：查看文件
cat docs/README.md
tail -f /var/log/messages  # 实时查看日志

# 场景5：清理练习
cd ~
rm -rf projects/
```

### 学习资源
- [Linux 目录结构详解](https://linuxjourney.com/lesson/the-directory-structure)
- [Linux 常用命令速查](https://linuxjourney.com/lesson/common-commands)

---

## 第2天：文本处理与查找命令

### 学习目标
- [ ] 掌握 grep、find、awk、sed 等文本处理命令
- [ ] 学会使用管道和重定向
- [ ] 理解通配符和正则表达式基础

### 知识点详解

#### 1. 查找命令

| 命令      | 功能                        | 示例                             |
| --------- | --------------------------- | -------------------------------- |
| `find`    | 查找文件                    | `find /home -name "*.log"`       |
| `grep`    | 文本搜索                    | `grep "error" /var/log/messages` |
| `which`   | 查找命令位置                | `which python`                   |
| `whereis` | 查找程序位置                | `whereis nginx`                  |
| `locate`  | 快速查找文件（需 updatedb） | `locate nginx.conf`              |

#### 2. 文本处理命令

| 命令   | 功能               | 示例                         |
| ------ | ------------------ | ---------------------------- |
| `awk`  | 文本处理语言       | `awk '{print $1}' file.txt`  |
| `sed`  | 流编辑器           | `sed 's/old/new/g' file.txt` |
| `cut`  | 切割文本           | `cut -d: -f1 /etc/passwd`    |
| `sort` | 排序               | `sort -n file.txt`           |
| `uniq` | 去重（需配合sort） | `sort file.txt \| uniq`      |
| `wc`   | 统计（行/词/字符） | `wc -l file.txt`             |

#### 3. 管道与重定向

```
>       # 输出重定向（覆盖）
>>      # 输出重定向（追加）
<       # 输入重定向
2>      # 错误输出重定向
2>&1    # 将错误输出合并到标准输出
|       # 管道，将前一个命令的输出作为后一个命令的输入
```

### 练习任务（5小时）

| 任务         | 预计时间 | 说明                       |
| ------------ | -------- | -------------------------- |
| grep 练习    | 1小时    | 在系统日志中搜索特定内容   |
| find 练习    | 1小时    | 查找系统中特定类型的文件   |
| 文本处理练习 | 1.5小时  | 使用 awk、sed 处理文本文件 |
| 管道练习     | 1小时    | 组合多个命令完成复杂任务   |
| 总结记录     | 0.5小时  | 记录常用场景和命令         |

### 练习场景

```bash
# 场景1：系统日志分析
grep -i "error" /var/log/messages                    # 搜索错误
grep -E "error|warning" /var/log/messages           # 正则搜索
grep -n "ssh" /var/log/messages                     # 显示行号
grep -c "error" /var/log/messages                    # 统计出现次数

# 场景2：文件查找
find /var/log -name "*.log" -mtime -7              # 查找7天内的日志文件
find /home -type f -size +100M                     # 查找大于100MB的文件
find /etc -name "*.conf" -exec grep "nginx" {} \;  # 查找配置文件中包含nginx的

# 场景3：文本处理
cut -d: -f1,7 /etc/passwd                          # 提取用户名和家目录
awk -F: '{print $1 "\t" $6}' /etc/passwd           # 格式化输出
sort -n -k3 /etc/passwd                            # 按UID排序
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10  # 统计访问IP前十

# 场景4：sed 替换
sed 's/old/new/g' file.txt                        # 全局替换
sed -i 's/old/new/g' file.txt                     # 直接修改文件
sed -n '10,20p' file.txt                          # 只显示第10-20行

# 场景5：管道组合
ps aux | grep nginx | grep -v grep                # 查看nginx进程
df -h | grep -vE "tmpfs|devtmpfs"                 # 过滤掉某些文件系统
netstat -tuln | grep LISTEN                        # 查看监听端口
```

### 学习资源
- [grep 教程](https://www.runoob.com/linux/linux-comm-grep.html)
- [awk 教程](https://www.runoob.com/linux/linux-comm-awk.html)

---

## 第3天：用户权限与系统管理

### 学习目标
- [ ] 理解 Linux 用户和权限模型
- [ ] 掌握用户、组、权限管理命令
- [ ] 理解 sudo 和 root 权限
- [ ] 学习 systemd 服务管理

### 知识点详解

#### 1. 权限模型

```
权限类型：
r = 4 = 读取
w = 2 = 写入
x = 1 = 执行

对象类型：
u = user（所有者）
g = group（所属组）
o = other（其他用户）
a = all（所有人）

示例：
-rwxr-xr--  user  group  file
  └─权限──── └──所有者── └─组── └─文件名
```

#### 2. 权限管理命令

| 命令    | 功能         | 示例                                   |
| ------- | ------------ | -------------------------------------- |
| `chmod` | 修改权限     | `chmod 755 file`、`chmod +x script.sh` |
| `chown` | 修改所有者   | `chown user:group file`                |
| `chgrp` | 修改所属组   | `chgrp group file`                     |
| `umask` | 设置默认权限 | `umask 022`                            |

#### 3. 用户管理命令

| 命令       | 功能     | 示例                          |
| ---------- | -------- | ----------------------------- |
| `useradd`  | 创建用户 | `useradd -m -s /bin/bash dev` |
| `userdel`  | 删除用户 | `userdel -r dev`              |
| `usermod`  | 修改用户 | `usermod -aG docker dev`      |
| `passwd`   | 修改密码 | `passwd dev`                  |
| `groupadd` | 创建组   | `groupadd developers`         |
| `gpasswd`  | 组管理   | `gpasswd -a dev developers`   |

#### 4. 系统管理命令

| 命令             | 功能             | 示例                    |
| ---------------- | ---------------- | ----------------------- |
| `systemctl`      | systemd 服务管理 | `systemctl start nginx` |
| `journalctl`     | 查看系统日志     | `journalctl -u nginx`   |
| `top`/`htop`     | 进程监控         | `top`                   |
| `ps`             | 查看进程         | `ps aux`                |
| `kill`/`killall` | 终止进程         | `kill -9 PID`           |

### 练习任务（5小时）

| 任务         | 预计时间 | 说明                          |
| ------------ | -------- | ----------------------------- |
| 权限练习     | 1.5小时  | 创建文件并练习各种权限操作    |
| 用户管理练习 | 1.5小时  | 创建用户、组，配置权限        |
| 服务管理练习 | 1小时    | 安装并管理一个服务（如nginx） |
| 进程管理练习 | 0.5小时  | 练习查看和终止进程            |
| 总结记录     | 0.5小时  | 记录常用场景                  |

### 练习场景

```bash
# 场景1：权限操作
touch testfile.txt
ls -l testfile.txt                # 查看权限
chmod 644 testfile.txt            # rw-r--r--
chmod +x testfile.txt             # 添加执行权限
chmod u+x,g+w,o-r testfile.txt    # 分别设置不同对象的权限

# 场景2：创建开发用户和组
groupadd developers
useradd -m -s /bin/bash -G developers devuser
passwd devuser                    # 设置密码

# 场景3：sudo 配置
visudo                            # 编辑sudoers文件
# 添加：devuser ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx

# 场景4：服务管理
yum install -y nginx              # CentOS
apt install -y nginx              # Ubuntu
systemctl start nginx
systemctl enable nginx            # 开机自启
systemctl status nginx
systemctl stop nginx

# 场景5：查看日志
journalctl -u nginx -f            # 实时查看nginx日志
journalctl -xe                    # 查看最近的错误日志

# 场景6：进程管理
ps aux | grep nginx               # 查看nginx进程
kill -HUP $(cat /var/run/nginx.pid)  # 平滑重启nginx
```

### 学习资源
- [Linux 权限详解](https://linuxjourney.com/lesson/file-permissions)
- [systemd 服务管理](https://www.freedesktop.org/software/systemd/man/systemd.html)

---

## 第4天：文件系统与Shell脚本入门

### 学习目标
- [ ] 理解 Linux 文件系统结构
- [ ] 掌握磁盘和存储管理命令
- [ ] 学习 Shell 脚本基础语法
- [ ] 编写第一个脚本

### 知识点详解

#### 1. 磁盘管理命令

| 命令     | 功能             | 示例                    |
| -------- | ---------------- | ----------------------- |
| `df`     | 查看磁盘使用情况 | `df -h`                 |
| `du`     | 查看目录大小     | `du -sh /var/log`       |
| `mount`  | 挂载文件系统     | `mount /dev/sdb1 /data` |
| `umount` | 卸载文件系统     | `umount /data`          |
| `fdisk`  | 磁盘分区         | `fdisk -l`              |
| `mkfs`   | 创建文件系统     | `mkfs.ext4 /dev/sdb1`   |

#### 2. 软链接与硬链接

```bash
# 硬链接：指向同一inode的多个文件名
ln file file-hardlink

# 软链接：指向文件路径的指针
ln -s file file-softlink

# 区别：
# - 硬链接不能跨文件系统，不能链接目录
# - 软链接可以跨文件系统，可以链接目录
# - 删除原文件后，硬链接仍可访问，软链接失效
```

#### 3. Shell 脚本基础

```bash
#!/bin/bash                    # 脚本开头，指定解释器

# 变量
name="devops"
echo "Hello, $name"

# 变量赋值不能有空格，使用时需要$

# 条件判断
if [ -f "/etc/passwd" ]; then
    echo "file exists"
fi

# 循环
for i in {1..10}; do
    echo "Number: $i"
done

# 函数
myfunc() {
    echo "This is a function"
}
myfunc
```

#### 4. 常用测试条件

```bash
# 文件测试
[ -f file ]    # 文件存在且为普通文件
[ -d dir ]     # 目录存在
[ -e path ]    # 路径存在
[ -r file ]    # 文件可读
[ -w file ]    # 文件可写
[ -x file ]    # 文件可执行

# 数值比较
[ $a -eq $b ]  # 相等
[ $a -ne $b ]  # 不相等
[ $a -gt $b ]  # 大于
[ $a -lt $b ]  # 小于

# 字符串比较
[ "$a" = "$b" ]    # 相等
[ "$a" != "$b" ]   # 不相等
[ -z "$a" ]        # 字符串为空
```

### 练习任务（5小时）

| 任务           | 预计时间 | 说明                       |
| -------------- | -------- | -------------------------- |
| 磁盘管理练习   | 1小时    | 使用虚拟机添加新磁盘并挂载 |
| 文件系统练习   | 0.5小时  | 练习软硬链接               |
| Shell 基础语法 | 1.5小时  | 学习变量、条件、循环       |
| 编写第一个脚本 | 1.5小时  | 完成练习任务               |
| 总结记录       | 0.5小时  | 记录笔记                   |

### 练习场景

```bash
# 场景1：查看磁盘使用情况
df -h                              # 查看磁盘使用
du -sh /var/log /var/lib           # 查看目录大小
du -h --max-depth=2 /var | sort -hr  # 查看最大的目录

# 场景2：软硬链接
touch original.txt
ln original.txt hardlink.txt       # 硬链接
ln -s original.txt softlink.txt    # 软链接
ls -li                             # 查看inode

# 场景3：编写备份脚本
#!/bin/bash
# backup.sh

SOURCE_DIR="/home/user/projects"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"

echo "Starting backup at $DATE..."
mkdir -p $BACKUP_DIR
tar -czf $BACKUP_FILE $SOURCE_DIR

echo "Backup completed: $BACKUP_FILE"
du -h $BACKUP_FILE

# 场景4：编写检查脚本
#!/bin/bash
# check_disk.sh

THRESHOLD=80
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | cut -d'%' -f1)

if [ $DISK_USAGE -gt $THRESHOLD ]; then
    echo "WARNING: Disk usage is ${DISK_USAGE}% (threshold: ${THRESHOLD}%)"
else
    echo "OK: Disk usage is ${DISK_USAGE}%"
fi

# 场景5：编写进程监控脚本
#!/bin/bash
# monitor_process.sh

PROCESS_NAME="nginx"

if pgrep -x $PROCESS_NAME > /dev/null; then
    echo "$PROCESS_NAME is running"
else
    echo "$PROCESS_NAME is not running"
    echo "Starting $PROCESS_NAME..."
    systemctl start $PROCESS_NAME
fi
```

### 学习资源
- [Bash 脚本教程](https://www.runoob.com/linux/linux-shell.html)
- [Shell 编程入门](https://linuxjourney.com/lesson/the-shell)

---

## 第5天：Linux 网络基础

### 学习目标
- [ ] 理解 TCP/IP 协议基础
- [ ] 掌握网络配置和排查命令
- [ ] 学会使用 SSH 远程连接
- [ ] 理解防火墙基础

### 知识点详解

#### 1. IP 地址和子网

```
IPv4 地址：4个字节，每个字节 0-255
示例：192.168.1.100

子网掩码：
- 255.255.255.0 (/24) 表示前24位是网络位
- 网络地址：IP & 掩码
- 广播地址：网络地址 | ~掩码
```

#### 2. 端口概念

```
常用端口：
22   - SSH
80   - HTTP
443  - HTTPS
3306 - MySQL
5432 - PostgreSQL
6379 - Redis
8080 - 应用常用端口
```

#### 3. 网络命令

| 命令         | 功能           | 示例                            |
| ------------ | -------------- | ------------------------------- |
| `ip`         | 网络配置       | `ip addr show`                  |
| `ifconfig`   | 网络配置（旧） | `ifconfig eth0`                 |
| `ping`       | 测试连通性     | `ping 8.8.8.8`                  |
| `traceroute` | 路由追踪       | `traceroute baidu.com`          |
| `netstat`    | 网络连接       | `netstat -tuln`                 |
| `ss`         | 网络连接（新） | `ss -tuln`                      |
| `curl`       | HTTP 请求      | `curl https://api.example.com`  |
| `wget`       | 下载文件       | `wget http://file.com/file.tar` |
| `ssh`        | 远程连接       | `ssh user@host`                 |
| `scp`        | 远程复制       | `scp file.txt user@host:/path`  |

#### 4. 防火墙基础

```bash
# firewalld (CentOS 7+)
firewall-cmd --state                    # 查看状态
firewall-cmd --list-ports              # 查看开放的端口
firewall-cmd --add-port=80/tcp --permanent  # 永久开放80端口
firewall-cmd --reload                  # 重载配置

# iptables
iptables -L -n                        # 查看规则
iptables -A INPUT -p tcp --dport 22 -j ACCEPT  # 开放22端口
```

### 练习任务（5小时）

| 任务         | 预计时间 | 说明                      |
| ------------ | -------- | ------------------------- |
| 网络命令练习 | 1.5小时  | 练习 ping、netstat、ss 等 |
| SSH 配置     | 1小时    | 配置 SSH 密钥登录         |
| 防火墙配置   | 1小时    | 配置防火墙规则            |
| 网络排查练习 | 1小时    | 模拟网络问题并排查        |
| 总结记录     | 0.5小时  | 记录笔记                  |

### 练习场景

```bash
# 场景1：查看网络配置
ip addr show                         # 查看所有网卡
ip route show                        # 查看路由表
cat /etc/resolv.conf                 # 查看DNS配置

# 场景2：网络连通性测试
ping -c 4 8.8.8.8                    # ping 4次
traceroute www.baidu.com             # 路由追踪
nslookup www.baidu.com              # DNS查询
dig www.baidu.com                    # DNS查询（详细）

# 场景3：查看网络连接
netstat -tuln                        # 查看监听端口
netstat -an                          # 查看所有连接
ss -tuln                             # 更快的连接查看
netstat -antp | grep nginx          # 查看nginx连接

# 场景4：SSH 配置
ssh-keygen -t rsa -b 4096           # 生成密钥
ssh-copy-id user@host                # 复制公钥到目标主机
ssh user@host                        # 密钥登录
# 编辑 /etc/ssh/sshd_config
# 禁用密码登录：PasswordAuthentication no

# 场景5：HTTP 测试
curl -I https://www.baidu.com       # 查看响应头
curl -X POST -d "data=test" http://api.example.com  # POST 请求
curl -o file.tar http://file.com/file.tar  # 下载文件

# 场景6：文件传输
scp file.txt user@host:/tmp/         # 上传文件
scp user@host:/tmp/file.txt ./      # 下载文件
scp -r dir/ user@host:/tmp/         # 上传目录
```

### 学习资源
- [TCP/IP 协议详解](https://www.runoob.com/tcpip/tcpip-intro.html)
- [SSH 教程](https://www.runoob.com/w3cnote/ssh-intro.html)

---

## 第6天：HTTP 协议与 Web 服务

### 学习目标
- [ ] 理解 HTTP/HTTPS 协议
- [ ] 掌握请求方法和状态码
- [ ] 学习 HTTP Header
- [ ] 部署和配置 Nginx

### 知识点详解

#### 1. HTTP 请求方法

| 方法    | 说明     | 示例用途           |
| ------- | -------- | ------------------ |
| GET     | 获取资源 | 获取网页、图片     |
| POST    | 提交数据 | 表单提交、创建资源 |
| PUT     | 更新资源 | 完整更新           |
| DELETE  | 删除资源 | 删除数据           |
| PATCH   | 部分更新 | 修改部分属性       |
| HEAD    | 获取头部 | 只检查资源是否存在 |
| OPTIONS | 预检请求 | CORS 预检          |

#### 2. HTTP 状态码

| 类别 | 状态码                    | 说明             |
| ---- | ------------------------- | ---------------- |
| 2xx  | 200 OK                    | 请求成功         |
|      | 201 Created               | 创建成功         |
|      | 204 No Content            | 成功但无返回内容 |
| 3xx  | 301 Moved Permanently     | 永久重定向       |
|      | 302 Found                 | 临时重定向       |
| 4xx  | 400 Bad Request           | 请求错误         |
|      | 401 Unauthorized          | 未认证           |
|      | 403 Forbidden             | 无权限           |
|      | 404 Not Found             | 资源不存在       |
| 5xx  | 500 Internal Server Error | 服务器错误       |
|      | 502 Bad Gateway           | 网关错误         |
|      | 503 Service Unavailable   | 服务不可用       |

#### 3. HTTP Header

```
请求头：
User-Agent: 客户端信息
Accept: 接受的内容类型
Authorization: 认证信息
Content-Type: 请求体类型

响应头：
Content-Type: 响应内容类型
Content-Length: 响应长度
Server: 服务器信息
Set-Cookie: 设置Cookie
Location: 重定向地址
```

#### 4. HTTPS

```
HTTP + SSL/TLS = HTTPS

特点：
- 数据加密传输
- 服务器身份认证
- 防止数据篡改

SSL/TLS 握手过程：
1. 客户端发送支持的加密算法
2. 服务器返回证书和选定的算法
3. 客户端验证证书，生成密钥
4. 服务器验证密钥
5. 开始加密通信
```

### 练习任务（5小时）

| 任务           | 预计时间 | 说明                   |
| -------------- | -------- | ---------------------- |
| HTTP 协议学习  | 1.5小时  | 学习请求方法、状态码   |
| Nginx 安装配置 | 1.5小时  | 安装并配置 Nginx       |
| HTTP 测试      | 1小时    | 使用 curl 测试各种场景 |
| HTTPS 基础     | 0.5小时  | 理解 HTTPS 原理        |
| 总结记录       | 0.5小时  | 记录笔记               |

### 练习场景

```bash
# 场景1：Nginx 安装
# CentOS
yum install -y nginx
systemctl start nginx
systemctl enable nginx

# Ubuntu
apt install -y nginx

# 场景2：HTTP 测试
curl -v https://www.baidu.com          # 详细输出
curl -I https://www.baidu.com          # 只看响应头
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"test"}' http://httpbin.org/post

# 场景3：测试各种状态码
curl https://httpbin.org/status/200
curl https://httpbin.org/status/404
curl https://httpbin.org/status/500

# 场景4：Nginx 配置
# 编辑 /etc/nginx/nginx.conf 或 /etc/nginx/conf.d/default.conf

server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    location /api {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# 重载配置
nginx -t                              # 检查配置
nginx -s reload                       # 重载配置

# 场景5：创建静态网站
mkdir /var/www/myapp
echo "Hello, World!" > /var/www/myapp/index.html

# 配置 Nginx
cat > /etc/nginx/conf.d/myapp.conf << EOF
server {
    listen 8080;
    server_name localhost;

    root /var/www/myapp;
    index index.html;
}
EOF

systemctl reload nginx
curl http://localhost:8080            # 测试访问
```

### 学习资源
- [HTTP 协议详解](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)
- [Nginx 官方文档](https://nginx.org/en/docs/)

---

## 第7天：DNS 解析与域名

### 学习目标
- [ ] 理解 DNS 解析原理
- [ ] 掌握 hosts 文件配置
- [ ] 学会使用 DNS 查询工具
- [ ] 理解 DNS 记录类型

### 知识点详解

#### 1. DNS 解析过程

```
1. 查看浏览器缓存
2. 查看系统缓存
3. 查看 hosts 文件
4. 查询本地 DNS 服务器
5. 查询根 DNS 服务器
6. 查询顶级域名 DNS 服务器
7. 查询权威 DNS 服务器
8. 返回 IP 地址
```

#### 2. DNS 记录类型

| 类型  | 说明                     | 示例                             |
| ----- | ------------------------ | -------------------------------- |
| A     | 域名到 IPv4 地址         | `www.example.com → 1.2.3.4`      |
| AAAA  | 域名到 IPv6 地址         | `www.example.com → ::1`          |
| CNAME | 域名别名                 | `www.example.com → example.com`  |
| MX    | 邮件服务器               | `example.com → mail.example.com` |
| TXT   | 文本记录（如 SPF、DKIM） |                                  |
| NS    | 名称服务器               | `example.com → ns1.example.com`  |
| PTR   | 反向解析                 | `1.2.3.4 → example.com`          |

#### 3. hosts 文件

```
位置：
- Linux/Mac: /etc/hosts
- Windows: C:\Windows\System32\drivers\etc\hosts

格式：IP地址  域名  别名
示例：
127.0.0.1    localhost
10.10.0.100  k8s-api.local
```

### 练习任务（5小时）

| 任务           | 预计时间 | 说明               |
| -------------- | -------- | ------------------ |
| DNS 原理学习   | 1小时    | 理解 DNS 解析过程  |
| DNS 查询工具   | 1小时    | 练习 nslookup、dig |
| hosts 文件配置 | 1小时    | 配置本地域名解析   |
| 实际场景练习   | 1.5小时  | 配置本地测试域名   |
| 总结记录       | 0.5小时  | 记录笔记           |

### 练习场景

```bash
# 场景1：DNS 查询
nslookup www.baidu.com               # 查询域名
nslookup -type=MX baidu.com          # 查询 MX 记录
dig www.baidu.com                    # 详细 DNS 信息
dig @8.8.8.8 www.baidu.com           # 指定 DNS 服务器查询

# 场景2：hosts 文件配置
cat /etc/hosts                       # 查看 hosts 文件
sudo vi /etc/hosts                   # 编辑 hosts 文件

# 添加以下内容
127.0.0.1    localhost
10.10.0.100  k8s-api.local
10.10.0.101  harbor.local
10.10.0.102  jenkins.local
10.10.0.103  grafana.local

# 场景3：测试本地解析
ping k8s-api.local
curl http://harbor.local

# 场景4：DNS 调试
# 查看当前 DNS 配置
cat /etc/resolv.conf

# 清除 DNS 缓存
# CentOS/RHEL
systemctl restart nscd
systemctl restart dnsmasq

# Ubuntu
sudo systemd-resolve --flush-caches

# 场景5：域名解析流程分析
dig +trace www.baidu.com            # 追踪 DNS 解析全过程
```

### 学习资源
- [DNS 基础教程](https://www.runoob.com/dns/dns-tutorial.html)
- [nslookup 和 dig 教程](https://www.runoob.com/linux/linux-comm-nslookup.html)

---

## 第8天：负载均衡原理

### 学习目标
- [ ] 理解负载均衡概念和原理
- [ ] 了解常见的负载均衡算法
- [ ] 学习四层和七层负载均衡
- [ ] 搭建简单的负载均衡环境

### 知识点详解

#### 1. 负载均衡类型

```
L4 负载均衡（传输层）：
- 基于 IP 地址和端口
- 工作在 TCP/UDP 层
- 性能高，但功能有限
- 如：HAProxy (mode tcp)、LVS

L7 负载均衡（应用层）：
- 基于 HTTP 请求内容
- 工作在应用层
- 功能丰富，可基于 URL、Cookie 等路由
- 如：HAProxy (mode http)、Nginx
```

#### 2. 常见负载均衡算法

| 算法              | 说明       | 适用场景       |
| ----------------- | ---------- | -------------- |
| Round Robin       | 轮询       | 服务器性能相近 |
| Least Connections | 最少连接   | 连接时间差异大 |
| Source IP         | 源地址哈希 | 需要会话保持   |
| URI Hash          | URL 哈希   | 缓存服务器     |
| Weighted          | 加权轮询   | 服务器性能不同 |

#### 3. 会话保持

```
方式：
1. 源地址哈希：同一IP请求同一后端
2. Cookie 插入：负载均衡器插入 Cookie
3. Cookie 重写：后端设置的 Cookie 重写
4. SSL Session ID：基于 SSL 会话
```

#### 4. 健康检查

```
检查方式：
- TCP 连接检查
- HTTP 响应检查（特定 URL 和状态码）
- 自定义脚本检查

作用：
- 自动剔除不健康节点
- 自动恢复健康节点
- 保证服务可用性
```

### 练习任务（5小时）

| 任务             | 预计时间 | 说明               |
| ---------------- | -------- | ------------------ |
| 负载均衡原理学习 | 1小时    | 理解概念和算法     |
| HAProxy 安装     | 1.5小时  | 安装并配置 HAProxy |
| 负载均衡测试     | 1.5小时  | 部署多个后端测试   |
| 健康检查配置     | 0.5小时  | 配置健康检查       |
| 总结记录         | 0.5小时  | 记录笔记           |

### 练习场景

```bash
# 场景1：安装 HAProxy
# CentOS
yum install -y haproxy

# Ubuntu
apt install -y haproxy

# 场景2：准备后端服务
# 启动两个 nginx 实例（不同端口）
docker run -d --name backend1 -p 8081:80 nginx
docker run -d --name backend2 -p 8082:80 nginx

# 场景3：配置 HAProxy
# /etc/haproxy/haproxy.cfg

global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

frontend myapp
    bind *:80
    default_backend backend_servers

backend backend_servers
    balance roundrobin
    server backend1 127.0.0.1:8081 check
    server backend2 127.0.0.1:8082 check

# 启动 HAProxy
systemctl start haproxy
systemctl enable haproxy

# 场景4：测试负载均衡
for i in {1..10}; do
  curl http://localhost
done

# 场景5：查看统计页面
# HAProxy 提供统计页面
# 添加到配置中：
listen stats
    bind *:8404
    stats enable
    stats uri /
    stats refresh 10s

# 访问 http://localhost:8404 查看统计信息

# 场景6：测试健康检查
# 停止一个后端服务
docker stop backend1

# 访问负载均衡，应该只路由到健康节点
curl http://localhost

# 恢复服务
docker start backend1
```

### 学习资源
- [HAProxy 官方文档](http://www.haproxy.org/#docs)
- [负载均衡算法详解](https://blog.csdn.net/qq_34409723/article/details/105692355)

---

## 第9天：综合练习与实战

### 学习目标
- [ ] 综合运用前8天所学知识
- [ ] 完成一个综合项目
- [ ] 加深对 Linux 系统的理解

### 综合项目：搭建简易 DevOps 基础环境

#### 项目需求
创建一个包含以下内容的简易环境：
1. Web 服务器（Nginx）
2. 后端服务（可以用简单的 Python Flask 或 Node.js）
3. 负载均衡（HAProxy）
4. 日志收集脚本
5. 监控脚本
6. 备份脚本

#### 项目步骤

**步骤1：准备目录结构**（30分钟）
```bash
mkdir -p ~/devops-project/{web,backend,scripts,logs,backup}
cd ~/devops-project
```

**步骤2：部署后端服务**（45分钟）
```bash
# 简单的 Python Flask 应用
cat > backend/app.py << 'EOF'
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return jsonify({
        'message': 'Hello from Backend',
        'host': os.getenv('HOSTNAME', 'unknown')
    })

@app.route('/health')
def health():
    return jsonify({'status': 'healthy'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# 运行两个后端实例
HOSTNAME=backend1 python3 backend/app.py &
HOSTNAME=backend2 python3 -c "
from backend.app import app
app.run(host='0.0.0.0', port=5001)
" &
```

**步骤3：配置负载均衡**（45分钟）
```bash
# 配置 HAProxy 轮询到两个后端
cat > /etc/haproxy/haproxy.cfg << 'EOF'
global
    log /dev/log local0
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    timeout connect 5000
    timeout client  50000
    timeout server  50000

frontend api
    bind *:8000
    default_backend backends

backend backends
    balance roundrobin
    server backend1 127.0.0.1:5000 check
    server backend2 127.0.0.1:5001 check

listen stats
    bind *:9000
    stats enable
    stats uri /
    stats refresh 5s
EOF

systemctl restart haproxy
```

**步骤4：配置前端**（30分钟）
```bash
# 创建简单的前端页面
cat > web/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>DevOps Project</title>
    <style>
        body { font-family: Arial; max-width: 800px; margin: 50px auto; }
        h1 { color: #333; }
        .result { background: #f0f0f0; padding: 20px; margin: 20px 0; }
        button { padding: 10px 20px; cursor: pointer; }
    </style>
</head>
<body>
    <h1>DevOps Learning Project</h1>
    <button onclick="callBackend()">Call Backend</button>
    <div id="result" class="result"></div>

    <script>
        function callBackend() {
            fetch('http://localhost:8000/')
                .then(r => r.json())
                .then(data => {
                    document.getElementById('result').innerHTML =
                        '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
                });
        }
    </script>
</body>
</html>
EOF

# 配置 Nginx
cat > /etc/nginx/conf.d/devops.conf << 'EOF'
server {
    listen 80;
    server_name localhost;

    location / {
        root /home/user/devops-project/web;
        index index.html;
    }

    location /api/ {
        proxy_pass http://localhost:8000/;
        proxy_set_header Host $host;
    }
}
EOF

systemctl restart nginx
```

**步骤5：编写日志收集脚本**（30分钟）
```bash
cat > scripts/collect_logs.sh << 'EOF'
#!/bin/bash
LOG_DIR="/home/user/devops-project/logs"
DATE=$(date +%Y%m%d)
mkdir -p $LOG_DIR

# 收集 Nginx 访问日志
cp /var/log/nginx/access.log $LOG_DIR/nginx_$DATE.log

# 收集系统日志
journalctl --since yesterday > $LOG_DIR/system_$DATE.log

echo "Logs collected at $(date)"
EOF

chmod +x scripts/collect_logs.sh
```

**步骤6：编写监控脚本**（30分钟）
```bash
cat > scripts/monitor.sh << 'EOF'
#!/bin/bash
LOG_DIR="/home/user/devops-project/logs"
DATE=$(date +%Y%m%d_%H%M%S)
REPORT="$LOG_DIR/monitor_$DATE.log"

echo "=== System Monitor Report ===" > $REPORT
echo "Time: $(date)" >> $REPORT
echo "" >> $REPORT

# CPU 使用率
echo "CPU Usage:" >> $REPORT
top -bn1 | grep "Cpu(s)" >> $REPORT
echo "" >> $REPORT

# 内存使用
echo "Memory Usage:" >> $REPORT
free -h >> $REPORT
echo "" >> $REPORT

# 磁盘使用
echo "Disk Usage:" >> $REPORT
df -h >> $REPORT
echo "" >> $REPORT

# 运行的服务
echo "Running Services:" >> $REPORT
systemctl list-units --type=service --state=running | grep -E "nginx|haproxy" >> $REPORT

echo "Report saved to $REPORT"
EOF

chmod +x scripts/monitor.sh
```

**步骤7：编写备份脚本**（30分钟）
```bash
cat > scripts/backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/home/user/devops-project/backup"
SOURCE_DIR="/home/user/devops-project"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"

mkdir -p $BACKUP_DIR

# 备份项目文件（排除日志和备份）
tar -czf $BACKUP_FILE \
    --exclude='logs' \
    --exclude='backup' \
    --exclude='__pycache__' \
    $SOURCE_DIR

# 保留最近7天的备份
find $BACKUP_DIR -name "backup_*.tar.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_FILE"
ls -lh $BACKUP_FILE
EOF

chmod +x scripts/backup.sh
```

**步骤8：配置定时任务**（30分钟）
```bash
# 编辑 crontab
crontab -e

# 添加以下内容
# 每天凌晨 2 点收集日志
0 2 * * * /home/user/devops-project/scripts/collect_logs.sh >> /home/user/devops-project/logs/cron.log 2>&1

# 每天凌晨 3 点运行监控
0 3 * * * /home/user/devops-project/scripts/monitor.sh >> /home/user/devops-project/logs/cron.log 2>&1

# 每天凌晨 4 点备份
0 4 * * * /home/user/devops-project/scripts/backup.sh >> /home/user/devops-project/logs/cron.log 2>&1

# 查看定时任务
crontab -l
```

**步骤9：测试整个环境**（30分钟）
```bash
# 访问前端
curl http://localhost

# 访问后端
curl http://localhost:8000/

# 测试负载均衡
for i in {1..5}; do curl http://localhost:8000/; echo; done

# 查看负载均衡统计
curl http://localhost:9000/

# 手动运行脚本
./scripts/collect_logs.sh
./scripts/monitor.sh
./scripts/backup.sh

# 查看日志
ls -lh logs/
```

### 练习任务（5小时）

| 任务         | 预计时间 | 说明                 |
| ------------ | -------- | -------------------- |
| 项目准备     | 0.5小时  | 创建目录结构         |
| 部署后端     | 0.75小时 | 部署 Flask 应用      |
| 配置负载均衡 | 0.75小时 | 配置 HAProxy         |
| 配置前端     | 0.5小时  | 配置 Nginx           |
| 编写脚本     | 1.5小时  | 日志、监控、备份脚本 |
| 配置定时任务 | 0.5小时  | 配置 crontab         |
| 测试验收     | 0.5小时  | 全面测试             |

### 学习成果
完成这个项目后，你将掌握：
- Linux 系统操作
- Web 服务部署
- 负载均衡配置
- Shell 脚本编写
- 系统监控
- 自动化任务

---

## 第10天：阶段总结与复盘

### 学习目标
- [ ] 回顾阶段一所有知识点
- [ ] 查漏补缺
- [ ] 准备进入阶段二的学习

### 复盘内容

#### 知识点自查表

| 知识点         | 掌握程度（0-10） | 需要加强的点 |
| -------------- | ---------------- | ------------ |
| Linux 常用命令 |                  |              |
| 文件权限管理   |                  |              |
| Shell 脚本     |                  |              |
| 网络基础       |                  |              |
| HTTP/HTTPS     |                  |              |
| DNS 解析       |                  |              |
| 负载均衡       |                  |              |
| 系统服务管理   |                  |              |

#### 常用命令速查

```bash
# 文件操作
ls, cd, pwd, mkdir, rm, cp, mv, touch

# 文本处理
grep, find, cat, less, head, tail, awk, sed

# 用户权限
chmod, chown, useradd, userdel, passwd

# 系统管理
systemctl, journalctl, ps, top, kill

# 网络操作
ip, ping, netstat, ssh, curl, wget

# 查看资源
df, du, free, htop

# 日志查看
tail -f, journalctl, cat /var/log/*/
```

### 练习任务（5小时）

| 任务           | 预计时间 | 说明                 |
| -------------- | -------- | -------------------- |
| 知识点回顾     | 1小时    | 浏览所有笔记和练习   |
| 命令速查表整理 | 1小时    | 整理个人命令速查表   |
| 薄弱知识点加强 | 2小时    | 针对薄弱点加强练习   |
| 阶段总结       | 1小时    | 写总结，规划下一阶段 |

### 阶段总结模板

```markdown
# 阶段一学习总结

## 学习时间
开始时间：____年__月__日
结束时间：____年__月__日
实际用时：___ 小时

## 掌握的知识点
1.
2.
3.

## 遇到的问题与解决方案
1. 问题：____
   解决方案：____

2. 问题：____
   解决方案：____

## 仍需加强的内容
1.
2.
3.

## 下阶段学习计划
目标：____
计划：____
```

---

## 学习资源汇总

### 在线学习平台
- [Linux Journey](https://linuxjourney.com/) - 免费 Linux 学习网站
- [OverTheWire](https://overthewire.org/wargames/) - Linux 游戏化学习
- [Katacoda](https://www.katacoda.com/) - 免费在线实验环境

### 推荐书籍
- 《鸟哥的Linux私房菜》- 入门经典
- 《Linux命令行与shell脚本编程大全》- 命令行和脚本
- 《深入理解计算机系统》- 深入理解

### 实用工具
- [tldr](https://tldr.sh/) - 命令简化版帮助
- [man 手册](https://linux.die.net/) - Linux 官方手册
- [explainshell.com](https://explainshell.com/) - 命令解释工具

---

## 进阶提示

完成阶段一后，你将具备进入下一阶段学习的基础条件：
- 可以使用命令行熟练操作 Linux
- 理解网络和 HTTP 基本原理
- 可以编写简单的 Shell 脚本
- 理解负载均衡的基本概念

下一阶段将学习 **Docker 容器技术**，建议在开始前：
1. 复习 Linux 基础命令
2. 确保虚拟机/云服务器正常运行
3. 准备好学习笔记和速查表

---

*祝你学习顺利！*
