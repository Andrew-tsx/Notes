# 阶段一：基础准备 - 详细学习计划 v2

> 学习周期：10天（2周）
> 每日学习时间：5小时（工作日3h碎片 + 晚间20:00-22:00连续2h）
> 开始日期：____年__月__日
> 完成日期：____年__月__日

---

## 可用资源

| 资源 | 用途 | 备注 |
|------|------|------|
| 工作笔记本 | 理论学习、查阅文档 | 工作时间碎片化学习 |
| 家中 Win10 PC | 晚间实操练习 | 可安装 WSL2 或虚拟机 |
| 飞牛 OS（家中） | 长期运行的实验环境 | 可部署服务、做实验 |
| RackNerd VPS（2C2G） | Linux 实操主战场 | 真实 Linux 环境，24h 可用 |

---

## 每日时间安排

| 时段 | 时长 | 内容类型 | 地点 |
|------|------|----------|------|
| 工作日碎片时间 | 约3h | 理论学习、阅读文档 | 工作笔记本 |
| 20:00-22:00 | 2h | 动手实操 | 家中 PC + VPS |

---

## 学习目标

- 熟练使用 Linux 常用命令
- 理解文件系统和权限管理
- 掌握网络基础知识
- 能够编写基础 Shell 脚本
- 理解 HTTP 协议和负载均衡原理

---

## 第1天：环境准备与文件操作

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 阅读Linux目录结构、常用命令文档 | 3h |
| 20:00-21:00 | SSH连接VPS，练习文件操作命令 | 1h |
| 21:00-22:00 | 练习场景 + 记录笔记 | 1h |

### 知识点

**Linux 目录结构：**
```
/                 # 根目录
├── bin/          # 常用命令
├── etc/          # 系统配置文件
├── home/         # 普通用户家目录
├── root/         # root用户家目录
├── var/          # 日志、缓存
├── usr/          # 用户程序
└── tmp/          # 临时文件
```

**常用命令：**
| 命令 | 功能 | 常用参数 |
|------|------|----------|
| ls | 列出文件 | -la, -h |
| cd | 切换目录 | -, ~ |
| pwd | 显示当前目录 | |
| mkdir | 创建目录 | -p |
| rm | 删除 | -r, -f |
| cp | 复制 | -r, -a |
| mv | 移动/重命名 | |
| touch | 创建空文件 | |
| cat | 查看文件 | |
| head/tail | 查看开头/结尾 | -n |

### 实操任务（VPS上执行）

```bash
# 1. SSH连接VPS
ssh root@你的VPS_IP

# 2. 创建项目目录结构
mkdir -p ~/projects/{dev,test,prod}
mkdir -p ~/projects/dev/{backend,frontend,docs}

# 3. 文件操作练习
cd ~/projects/dev
touch app.py config.yaml README.md
ls -la

# 4. 复制和移动
cp app.py ../test/
mv README.md docs/

# 5. 查看文件
cat docs/README.md
tail -f /var/log/syslog  # 实时查看日志
```

### 学习资源（免费）
- [Linux命令大全](https://www.runoob.com/linux/linux-command-manual.html)
- [鸟哥的Linux私房菜](https://linux.vbird.org/)
- [菜鸟教程 Linux](https://www.runoob.com/linux/linux-tutorial.html)

---

## 第2天：文本处理与查找

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 学习grep、find、awk、sed | 3h |
| 20:00-21:00 | VPS上练习文本处理命令 | 1h |
| 21:00-22:00 | 管道组合练习 + 笔记 | 1h |

### 知识点

| 命令 | 功能 | 示例 |
|------|------|------|
| find | 查找文件 | `find /home -name "*.log"` |
| grep | 文本搜索 | `grep "error" /var/log/syslog` |
| awk | 文本处理 | `awk '{print $1}' file.txt` |
| sed | 流编辑器 | `sed 's/old/new/g' file.txt` |
| cut | 切割文本 | `cut -d: -f1 /etc/passwd` |
| sort | 排序 | `sort -n file.txt` |
| wc | 统计 | `wc -l file.txt` |

**管道与重定向：**
```
>     # 输出重定向（覆盖）
>>    # 输出重定向（追加）
|     # 管道
2>&1  # 错误合并到标准输出
```

### 实操任务

```bash
# 1. grep练习
grep -i "error" /var/log/syslog
grep -E "error|warning" /var/log/syslog

# 2. find练习
find /var/log -name "*.log" -mtime -7
find /home -type f -size +10M

# 3. 文本处理
cut -d: -f1 /etc/passwd
awk -F: '{print $1}' /etc/passwd
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# 4. sed替换
sed 's/old/new/g' test.txt
```

### 学习资源
- [grep教程](https://www.runoob.com/linux/linux-comm-grep.html)
- [awk教程](https://www.runoob.com/linux/linux-comm-awk.html)
- [sed教程](https://www.runoob.com/linux/linux-comm-sed.html)

---

## 第3天：用户权限与系统管理

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 学习权限模型、用户管理、systemd | 3h |
| 20:00-21:00 | VPS上练习用户和权限管理 | 1h |
| 21:00-22:00 | 安装nginx服务 + 练习systemctl | 1h |

### 知识点

**权限模型：**
```
r = 4, w = 2, x = 1
u=user, g=group, o=other, a=all

示例：-rwxr-xr--
      所有者rwx(7)，组r-x(5)，其他r--(4)
```

**常用命令：**
| 命令 | 功能 | 示例 |
|------|------|------|
| chmod | 修改权限 | `chmod 755 file` |
| chown | 修改所有者 | `chown user:group file` |
| useradd | 创建用户 | `useradd -m dev` |
| passwd | 修改密码 | `passwd dev` |
| systemctl | 服务管理 | `systemctl start nginx` |
| journalctl | 查看日志 | `journalctl -u nginx` |

### 实操任务

```bash
# 1. 权限练习
touch testfile.txt
chmod 644 testfile.txt
chmod +x testfile.txt

# 2. 创建用户
useradd -m -s /bin/bash devuser
passwd devuser
usermod -aG sudo devuser  # Ubuntu

# 3. 安装nginx
apt update && apt install -y nginx
systemctl start nginx
systemctl enable nginx
systemctl status nginx

# 4. 查看日志
journalctl -u nginx -f
```

### 学习资源
- [Linux权限详解](https://www.runoob.com/linux/linux-file-attr-permission.html)
- [systemd入门](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

---

## 第4天：文件系统与Shell脚本入门

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 学习磁盘管理、Shell语法 | 3h |
| 20:00-21:00 | 练习磁盘命令和链接 | 1h |
| 21:00-22:00 | 编写第一个Shell脚本 | 1h |

### 知识点

**磁盘命令：**
| 命令 | 功能 | 示例 |
|------|------|------|
| df | 查看磁盘使用 | `df -h` |
| du | 查看目录大小 | `du -sh /var` |
| mount | 挂载文件系统 | `mount /dev/sdb1 /data` |

**软链接与硬链接：**
```bash
ln file hardlink      # 硬链接
ln -s file softlink   # 软链接
```

**Shell脚本基础：**
```bash
#!/bin/bash
name="devops"
echo "Hello, $name"

if [ -f "/etc/passwd" ]; then
    echo "file exists"
fi

for i in {1..5}; do
    echo "Number: $i"
done
```

### 实操任务

```bash
# 1. 查看磁盘
df -h
du -sh /var/log

# 2. 创建链接
touch original.txt
ln original.txt hardlink.txt
ln -s original.txt softlink.txt
ls -li

# 3. 编写备份脚本
cat > ~/backup.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
echo "Backup started at $DATE"
tar -czf ~/backup_$DATE.tar.gz ~/projects
echo "Backup completed"
EOF
chmod +x ~/backup.sh
./backup.sh
```

### 学习资源
- [Bash脚本教程](https://www.runoob.com/linux/linux-shell.html)
- [Shell脚本入门](https://www.runoob.com/linux/linux-shell-process.html)

---

## 第5天：Linux网络基础

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 学习TCP/IP、端口、网络命令 | 3h |
| 20:00-21:00 | VPS上练习网络命令 | 1h |
| 21:00-22:00 | SSH密钥配置 + 防火墙基础 | 1h |

### 知识点

**常用端口：**
```
22   - SSH
80   - HTTP
443  - HTTPS
3306 - MySQL
6379 - Redis
```

**网络命令：**
| 命令 | 功能 | 示例 |
|------|------|------|
| ip | 网络配置 | `ip addr` |
| ping | 测试连通性 | `ping 8.8.8.8` |
| ss | 查看连接 | `ss -tuln` |
| curl | HTTP请求 | `curl -I baidu.com` |
| ssh | 远程连接 | `ssh user@host` |
| scp | 远程复制 | `scp file user@host:/path` |

### 实操任务

```bash
# 1. 查看网络配置
ip addr show
ip route show
cat /etc/resolv.conf

# 2. 网络测试
ping -c 4 8.8.8.8
ss -tuln

# 3. SSH密钥配置（在本地PC执行）
ssh-keygen -t rsa -b 4096
ssh-copy-id root@VPS_IP

# 4. HTTP测试
curl -I https://www.baidu.com
curl -o test.html https://www.baidu.com
```

### 学习资源
- [TCP/IP入门](https://www.runoob.com/tcpip/tcpip-intro.html)
- [SSH教程](https://www.runoob.com/w3cnote/ssh-intro.html)

---

## 第6天：HTTP协议与Web服务

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 学习HTTP协议、状态码、Header | 3h |
| 20:00-21:00 | VPS上配置nginx | 1h |
| 21:00-22:00 | 用curl测试HTTP + 配置虚拟主机 | 1h |

### 知识点

**HTTP状态码：**
| 类别 | 常见状态码 | 含义 |
|------|-----------|------|
| 2xx | 200, 201 | 成功 |
| 3xx | 301, 302 | 重定向 |
| 4xx | 400, 401, 403, 404 | 客户端错误 |
| 5xx | 500, 502, 503 | 服务端错误 |

**HTTPS原理：**
```
HTTP + SSL/TLS = HTTPS
- 数据加密传输
- 服务器身份认证
- 防止数据篡改
```

### 实操任务

```bash
# 1. HTTP测试
curl -v https://www.baidu.com
curl -I https://www.baidu.com
curl -X POST -d "name=test" http://httpbin.org/post

# 2. nginx配置（VPS上）
# 编辑 /etc/nginx/sites-available/default
# 添加自定义配置

# 3. 创建静态站点
mkdir -p /var/www/myapp
echo "Hello DevOps!" > /var/www/myapp/index.html

# 4. 配置nginx虚拟主机
cat > /etc/nginx/sites-available/myapp << 'EOF'
server {
    listen 8080;
    root /var/www/myapp;
    index index.html;
}
EOF
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
curl http://localhost:8080
```

### 学习资源
- [HTTP协议详解](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)
- [nginx入门](https://www.runoob.com/w3cnote/nginx-setup-intro.html)

---

## 第7天：DNS解析与域名

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 学习DNS原理、记录类型 | 3h |
| 20:00-21:00 | 练习DNS查询工具 | 1h |
| 21:00-22:00 | 配置hosts文件 + 本地域名解析 | 1h |

### 知识点

**DNS记录类型：**
| 类型 | 说明 |
|------|------|
| A | 域名→IPv4 |
| CNAME | 别名 |
| MX | 邮件服务器 |
| TXT | 文本记录 |

**hosts文件：**
```
Linux: /etc/hosts
Windows: C:\Windows\System32\drivers\etc\hosts

格式：IP地址  域名
示例：10.10.0.100  k8s-api.local
```

### 实操任务

```bash
# 1. DNS查询
nslookup www.baidu.com
dig www.baidu.com
dig @8.8.8.8 www.baidu.com

# 2. 配置hosts
echo "127.0.0.1 myapp.local" >> /etc/hosts
ping myapp.local

# 3. 配置测试域名（模拟内网环境）
cat >> /etc/hosts << 'EOF'
# DevOps学习测试域名
10.10.0.100  k8s-api.local
10.10.0.101  harbor.local
10.10.0.102  jenkins.local
10.10.0.103  grafana.local
EOF

# 4. DNS追踪
dig +trace www.baidu.com
```

### 学习资源
- [DNS入门](https://www.runoob.com/dns/dns-tutorial.html)
- [dig命令详解](https://www.runoob.com/linux/linux-comm-dig.html)

---

## 第8天：负载均衡原理

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 学习负载均衡概念、算法、L4/L7 | 3h |
| 20:00-21:00 | VPS上安装配置HAProxy | 1h |
| 21:00-22:00 | 测试负载均衡 + 健康检查 | 1h |

### 知识点

**负载均衡类型：**
```
L4（传输层）：基于IP+端口，如 HAProxy TCP模式
L7（应用层）：基于HTTP内容，如 Nginx、HAProxy HTTP模式
```

**负载均衡算法：**
| 算法 | 说明 |
|------|------|
| Round Robin | 轮询 |
| Least Connections | 最少连接 |
| Source IP Hash | 源地址哈希 |

### 实操任务

```bash
# 1. 安装HAProxy
apt install -y haproxy

# 2. 准备后端（用nginx不同端口模拟）
# 先在8081和8082启动两个简单服务

# 3. 配置HAProxy
cat > /etc/haproxy/haproxy.cfg << 'EOF'
global
    log /dev/log local0

defaults
    log global
    mode http
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend myapp
    bind *:80
    default_backend servers

backend servers
    balance roundrobin
    server s1 127.0.0.1:8081 check
    server s2 127.0.0.1:8082 check
EOF

# 4. 启动测试
systemctl restart haproxy
for i in {1..10}; do curl http://localhost; done
```

### 学习资源
- [HAProxy入门](https://www.haproxy.org/)
- [负载均衡原理](https://www.runoob.com/w3cnote/load-balancing-intro.html)

---

## 第9天：综合实战

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 回顾前8天内容，规划项目 | 3h |
| 20:00-22:00 | 完成综合项目（2h连续） | 2h |

### 综合项目：简易DevOps环境

**目标：** 在VPS上搭建一个包含Nginx + 负载均衡 + 监控脚本的简易环境

```bash
# 步骤1：创建目录结构
mkdir -p ~/devops-project/{web,scripts,logs}

# 步骤2：创建测试页面
echo '<h1>Backend 1</h1>' > /var/www/html/backend1.html
echo '<h1>Backend 2</h1>' > /var/www/html/backend2.html

# 步骤3：配置nginx两个端口
# 编辑nginx配置，监听8081和8082

# 步骤4：配置HAProxy负载均衡
# 参考第8天内容

# 步骤5：编写监控脚本
cat > ~/devops-project/scripts/monitor.sh << 'EOF'
#!/bin/bash
echo "=== System Monitor $(date) ==="
echo "CPU: $(top -bn1 | grep 'Cpu' | awk '{print $2}')%"
echo "Memory: $(free -m | awk 'NR==2{print $3"/"$2"MB"}')"
echo "Disk: $(df -h / | awk 'NR==2{print $5}')"
echo "Nginx: $(systemctl is-active nginx)"
echo "HAProxy: $(systemctl is-active haproxy)"
EOF
chmod +x ~/devops-project/scripts/monitor.sh

# 步骤6：测试
curl http://localhost
~/devops-project/scripts/monitor.sh
```

---

## 第10天：总结与复盘

### 时间安排

| 时段 | 内容 | 时长 |
|------|------|------|
| 工作日 | 回顾所有笔记，查漏补缺 | 3h |
| 20:00-21:00 | 整理命令速查表 | 1h |
| 21:00-22:00 | 写阶段总结，规划下一阶段 | 1h |

### 知识点自查表

| 知识点 | 掌握程度(0-10) | 需加强 |
|--------|----------------|--------|
| Linux常用命令 | | |
| 文件权限管理 | | |
| Shell脚本 | | |
| 网络基础 | | |
| HTTP/HTTPS | | |
| DNS解析 | | |
| 负载均衡 | | |
| 系统服务管理 | | |

### 命令速查表

```bash
# 文件操作
ls, cd, pwd, mkdir, rm, cp, mv, touch, cat, head, tail

# 文本处理
grep, find, awk, sed, cut, sort, uniq, wc

# 用户权限
chmod, chown, useradd, userdel, passwd, groups

# 系统管理
systemctl, journalctl, ps, top, kill, df, du

# 网络
ip, ping, ss, curl, wget, ssh, scp

# 服务
nginx -t, nginx -s reload, haproxy -c
```

### 阶段总结模板

```markdown
# 阶段一学习总结

## 学习时间
开始：____  结束：____  实际用时：____h

## 已掌握
1.
2.

## 遇到的问题
1. 问题：____  解决：____

## 需加强
1.

## 下一阶段计划
```

---

## 学习资源汇总（全部免费）

### 在线文档
- [菜鸟教程 Linux](https://www.runoob.com/linux/linux-tutorial.html)
- [鸟哥的Linux私房菜](https://linux.vbird.org/)
- [Runoob Shell教程](https://www.runoob.com/linux/linux-shell.html)
- [MDN HTTP文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

### 实用工具
- [explainshell.com](https://explainshell.com/) - 命令解释
- [tldr](https://tldr.sh/) - 简化版man手册

### 书籍（可选）
- 《鸟哥的Linux私房菜》
- 《Linux命令行与shell脚本编程大全》

---

## 进阶提示

完成阶段一后，你将具备：
- 命令行操作Linux的能力
- 网络和HTTP基础知识
- 编写简单Shell脚本的能力
- 负载均衡基本概念

**下一阶段：Docker容器技术**

---

*最后更新：2026-03-26*
