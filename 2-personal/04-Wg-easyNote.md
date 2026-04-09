# install wg-easy
**官方网址**
https://wg-easy.github.io/wg-easy/v15.2/examples/tutorials/basic-installation/
**Github项目地址**
https://github.com/wg-easy/wg-easy/blob/master/README.md

**Create a directory for the configuration files**
sudo mkdir -p /etc/docker/containers/wg-easy

**Download docker compose file**
sudo curl -o /etc/docker/containers/wg-easy/docker-compose.yml https://raw.githubusercontent.com/wg-easy/wg-easy/master/docker-compose.yml

**start wg-easy**
cd /etc/docker/containers/wg-easy
sudo docker compose up -d

**访问管理面板**
默认是https://IP:51821，初始化之后登陆时会提示禁止使用https登录
更改
```
- PORT=6660
- INSECURE=true
```
访问地址更换为

面板端口6660
udp端口6670

**查询docker内部的wg文件路径**
docker volume inspect wg-easy_etc_wireguard
/var/lib/docker/volumes/wg-easy_etc_wireguard/_data



wireguard绕过封锁指南
https://www.ctbots.com/zh/blog/net/wireguard/bypassBlock.html

iptables -t nat -A POSTROUTING -s {{ipv4Cidr}} -o {{device}} -j MASQUERADE; iptables -A INPUT -p udp -m udp --dport {{port}} -j ACCEPT; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -s {{ipv6Cidr}} -o {{device}} -j MASQUERADE; ip6tables -A INPUT -p udp -m udp --dport {{port}} -j ACCEPT; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -A FORWARD -o wg0 -j ACCEPT;


### 先搞懂基础概念（小白必看）
在解释具体命令前，先明确3个核心认知：
1. **WireGuard的钩子命令**：`PreUp/PostUp/PreDown/PostDown` 是WireGuard的“触发指令”——
   - `PreUp`：wg0（VPN虚拟网卡）启动**前**要执行的命令（你这里是空的，啥也不做）；
   - `PostUp`：wg0启动**后**要执行的命令（核心是配置VPN上网/端口放行）；
   - `PreDown`：wg0停止**前**要执行的命令（你这里是空的）；
   - `PostDown`：wg0停止**后**要执行的命令（删除PostUp加的规则，避免残留）。
2. **关键“硬件”比喻**：
   - `eth0`：VPS的“公网网线”（连互联网的真实网卡）；
   - `wg0`：VPN的“虚拟网线”（只有连了VPN的设备才会用这个网线）；
   - `iptables`：Linux的“流量管理员”（管哪些流量能进/出/转发），`ip6tables`是IPv6版本的“管理员”。
3. **核心动作比喻**：
   - `-A`：给“管理员”加新规则；`-D`：删除之前加的规则；
   - `MASQUERADE`：“伪装”——让VPN客户端的流量“披着VPS的公网IP”上网（不然客户端没法访问外网）。

---

## 一、PostUp（VPN启动后执行的规则，核心是让VPN能上网+放行端口）
PostUp里的命令是“分号;”分隔的，拆成8条逐一解释：

### 1. `iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE`
- **大白话**：让所有来自VPN客户端（IP段10.8.0.0/24）的流量，从VPS的公网网卡eth0出去时，“伪装”成VPS的公网IP。
- **为啥要做**：如果不伪装，互联网上的网站会收到“来自10.8.0.x”的流量，不认识这个IP，会直接拒绝，VPN客户端就上不了网。

### 2. `iptables -A INPUT -p udp -m udp --dport 6750 -j ACCEPT`
- **大白话**：允许外部设备通过UDP 6750端口（你改后的VPN数据端口）连接到VPS（就是让VPN客户端能连上来）。
- **为啥要做**：如果不加这个规则，VPS的“管理员”会把VPN客户端的连接请求拦下来，客户端连不上VPN。

### 3. `iptables -A FORWARD -i wg0 -j ACCEPT`
- **大白话**：允许从VPN虚拟网卡wg0进来的流量，通过VPS转发到其他地方（比如互联网）。
- **为啥要做**：VPN客户端的流量先到wg0网卡，VPS需要“转发”这个流量到eth0公网卡，才能上网。

### 4. `iptables -A FORWARD -o wg0 -j ACCEPT`
- **大白话**：允许从互联网来的流量，通过VPS转发到VPN虚拟网卡wg0（就是让VPN客户端能收到网站的返回数据）。
- **为啥要做**：网站返回的流量先到eth0公网卡，VPS需要转发到wg0，客户端才能收到数据（比如打开网页能看到内容）。

### 5. `ip6tables -t nat -A POSTROUTING -s fdcc:ad94:bacf:61a4::cafe:0/112 -o eth0 -j MASQUERADE`
- **大白话**：和第1条一样，只是针对IPv6的VPN客户端（IP段fdcc:ad94:bacf:61a4::cafe:0/112），让它们的IPv6流量伪装成VPS的IPv6地址上网。
- **为啥要做**：你禁用了IPv6，这条规则实际没用，但wg-easy自动生成了，不影响使用。

### 6. `ip6tables -A INPUT -p udp -m udp --dport 6750 -j ACCEPT`
- **大白话**：允许IPv6设备通过UDP 6750端口连接VPN。
- **为啥要做**：同样因为你禁用了IPv6，这条规则没用，只是自动生成的。

### 7. `ip6tables -A FORWARD -i wg0 -j ACCEPT`
- **大白话**：允许IPv6流量从wg0进来后转发。
- **为啥要做**：IPv6禁用，没用。

### 8. `ip6tables -A FORWARD -o wg0 -j ACCEPT`
- **大白话**：允许IPv6流量转发到wg0。
- **为啥要做**：IPv6禁用，没用。

---

## 二、PostDown（VPN停止后执行的规则，核心是“清理垃圾”）
PostDown里的命令和PostUp一一对应，只是把 `-A`（添加）改成了 `-D`（删除），目的是：
VPN停止后，删掉之前加的所有规则，避免这些规则残留，导致VPS的防火墙/网络出问题。

逐条解释（和PostUp反向操作）：
1. `iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE` → 删除IPv4流量伪装规则；
2. `iptables -D INPUT -p udp -m udp --dport 6750 -j ACCEPT` → 删除UDP 6750端口放行规则；
3. `iptables -D FORWARD -i wg0 -j ACCEPT` → 删除wg0进来的转发规则；
4. `iptables -D FORWARD -o wg0 -j ACCEPT` → 删除wg0出去的转发规则；
5-8. 删掉IPv6相关的所有规则（同样因为你禁用了IPv6，删不删都无所谓）。

---

## 三、小白重点总结
1. **核心有用的规则是前4条IPv4规则**：作用是“让VPN客户端能连上来、能上网、能收到网站数据”；
2. **后4条IPv6规则没用**：因为你在docker-compose里禁用了IPv6，可忽略；
3. **PostDown是“善后”**：VPN关了就删掉这些规则，避免VPS网络混乱；
4. **如果改了VPN端口（比如从6750改成其他）**：这些规则里的`--dport 6750`会自动改成新端口，不用手动改。

简单说，这些规则就是给VPN“铺路”——让客户端的流量能从VPS的公网网卡出去，也能从公网网卡回来，没有这些规则，VPN只能连得上，但上不了网。


# 自动获取公网IP
VPS_PUBLIC_IP=$(curl -4 icanhazip.com)

docker run -d \
  --name=wg-easy \
  -e LANG=zh_CN \
  -e WG_HOST=$VPS_PUBLIC_IP \
  -e PASSWORD_HASH=$(docker run --rm ghcr.io/wg-easy/wg-easy:14 node -e "console.log(require('bcryptjs').hashSync('$MY_PASS', 10))") \
  -e WG_PORT=56655 \
  -e PORT=58080 \
  -v /etc/wireguard:/etc/wireguard \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  -p 56655:56655/udp \
  -p 58080:58080/tcp \
  --restart unless-stopped \
  ghcr.io/wg-easy/wg-easy:14


# 产生hash_password
docker run --rm -it ghcr.io/wg-easy/wg-easy:14 wgpw Szst@321
$2a$12$tCxlHaOZwC73cYi2SmbgsuKzNsICe3b4oviO/Yy5ktg80GDc35Epa

$2y$12$ig78KEqF121XcwQBD6uE8etEdSBA6ydPF/.NAjGk3JeBIxHJ7wsmK

$2a$12$T4vFPhLY7j.0YIvG3CLBTOvJGckaLXGSKzdW9d9BqC/LDSZmWBDuK

docker run ghcr.io/wg-easy/wg-easy:14 node -e 'const bcrypt = require("bcryptjs"); const hash = bcrypt.hashSync("Szst@321", 10); console.log(hash.replace(/\$/g, "$$$$"));'

docker run --rm ghcr.io/wg-easy/wg-easy:14 node -e "const bcrypt = require('bcryptjs'); const hash = bcrypt.hashSync('Szst@321', 10); console.log('\'' + hash + '\'');"
$2a$10$tFcMzvGf5jrvxw6a0WxTNuNxH3MYjWwHpRhdKeDn8ghf3CGCk.rxu

https://blog.csdn.net/Hu560231/article/details/158889365

https://blog.mvpbang.com/p/9b629de3fa70462fa7123aa35c2844b3/



