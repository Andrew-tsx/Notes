# namesilo
namesilo优惠码
TYVKVQ
ESY
Andrew Chou
5/23/2003
3835 Columbia Road, Philadelphia, Delaware, 19103
302-519-1480
**unclechou.top**
nameserver
walk.ns.cloudflare.com
maciej.ns.cloudflare.com


# DigitalPlat Domains
Cassin Roland
zhouweiqing0yuzhe@gmail.com
BcJ88v0aFzQD0igGinAg
+1-3029039904
101 Avenue of the Arts, Wilmington, Delaware 19801, United States

andrew.dpdns.org
unclechou.dpdns.org

fred.ns.cloudflare.com
magali.ns.cloudflare.com





# FNos部署v2raya
version: '3.8'             # docker-compose 文件语法版本

services:
  v2raya:                  # 服务名字（随便起）
    image: mzz2017/v2raya  # 镜像名，来自 docker hub
    restart: always        # 自动重启策略，容器挂了会重新拉起
    container_name: v2raya # 容器名字，等价于 `--name v2raya`
    
    network_mode: host     # 使用宿主机网络（端口直接绑定到宿主机）
    privileged: true       # 提升权限，方便操作 iptables 等

    cap_add:               # 增加额外能力（比 privileged 更细粒度）
      - NET_ADMIN          # 网络管理权限
      - SYS_MODULE         # 加载内核模块权限
    volumes:
      - ./etc:/etc/v2raya  # 把当前目录下的 ./etc 映射到容器里的 /etc/v2raya
                           # 用来存放 v2raya 的配置文件，重启不会丢




# 部署面板
1. 更新软件包
`apt update -y && apt upgrade -y && apt install sudo curl -y`
2. 关闭ufw防火墙
`systemctl stop ufw.service`
3. 安装3XUI面板
`bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)`
4. 记录面板信息
═══════════════════════════════════════════
Username:    admin
Password:    Wwf5y6Ce4jdv6ZY0p6BM
Port:        56650
WebBasePath: vAGEy9kAgA8CPAu3wZ
Access URL:  https://107.175.54.178:56650/vAGEy9kAgA8CPAu3wZ
https://learn.unclechou.top:8443/vAGEy9kAgA8CPAu3wZ/
═══════════════════════════════════════════
5. 开启mbr算法

6. 登录面板，入站列表
根据协议、传输、安全三个地方来配置节点
- vless + XHTTP + Reality
- vless + tcp + Reality
- vmess + WebSocket + TLS
若选择TLS加密，则流量类型为https，cloudflare针对https只开放
443, 2053, 2083, 2087, 2096, 8443
主机：learn.unclechou.top，连接cdn
SNI：learn.unclechou.top，伪装欺骗gfw 

7. CloudFlare解析域名
**learn.unclechou.top**
cdn加速

8. x-ui安装SSL证书，绑定域名
[INF] Panel paths set for domain: learn.unclechou.top 
[INF]   - Certificate File: /root/cert/learn.unclechou.top/fullchain.pem 
[INF]   - Private Key File: /root/cert/learn.unclechou.top/privkey.pem 
Access URL: https://learn.unclechou.top:56650/vAGEy9kAgA8CPAu3wZ/
[INF] x-ui and xray Restarted successfully 
注意，开启了CF cdn加速后该网址无法访问，原因在于端口被CF限制


9. ip优选
CloudflareSpeedTest
Github仓库地址
https://github.com/XIU2/CloudflareSpeedTest
cloudflaresub
Github仓库地址
https://github.com/InfiCheesy/cloudflaresub

1. 根据视频Fork仓库，部署Worker
2. 设置KV命名空间
```powershell
SUB_STORE
```
3. 「变量和机密」添加「密钥」
```powershell
SUB_ACCESS_TOKEN
```
4. my替换器
https://cftest.uncle-andrew.workers.dev/


