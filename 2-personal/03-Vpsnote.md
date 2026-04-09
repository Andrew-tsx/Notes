# VPS Settings

## server detail

*2 GB KVM VPS (New Year Special) (Server Label: racknerd-1a585a3)
IP Address: 107.175.50.150
LocationLos Angeles DC03 (Test IP: 107.174.51.158)
Username: root
Root Password: Iej00OEG4ZP8ho31vz
SSH Port 22
优惠码
15OFFDEDI*

**RackNerd**
107.175.54.178
服务器后台控制页面
<https://nerdvm.racknerd.com/>


**Aliyun**
139.224.3.240
172.31.182.183


## 解决xshell无法连接vps，报错host无法解密

`vi /etc/ssh/sshd.conf`
在文件最后加入下列参数

```bash
HostKeyAlgorithms +ssh-rsa,rsa-sha2-256,rsa-sha2-512,ssh-ed25519
PubkeyAcceptedAlgorithms +ssh-rsa,rsa-sha2-256,rsa-sha2-512,ssh-ed25519
```

## 远程端口、密码、防火墙、hostname

1. **远程端口**
    `vi /etc/ssh/sshd.conf`
    修改Port参数，注意一定不能删除原有的22端口！
    建议只在最后增加下列两行

    ```bash
    Port 22
    Port your_port_number
    ```

    Ubuntu 24版本下，配置文件有注释提示
    ssh.socket 在 daemon-reload 后会重新读取 sshd_config 里的 Port 配置并生成对应的 socket 监听。
    修改完毕执行下列命令即可

    ```bash
    systemctl daemon-reload
    systemctl restart ssh.socket
    ```

    不要用 systemctl restart sshd 或 systemctl restart ssh，在 socket activation 模式下那一步不够。

2. **修改root用户密码**
    `passwd`
    之后输入两次密码，不会在界面显示

3. **关闭ufw防火墙**
    先将防火墙关闭，防止后续无法远程上去
    等远程端口修改生效后，增加防火墙规则，再开启防火墙
    `systemctl stop ufw.service`

4. **修改hostname**
    默认hostname很长，可用下列命令改成想要的名字
    `hostnamectl set-hostname your_name`

## 解决远程端口更改后不生效的问题

密钥权限被意外修改（比如 chmod 777）或密钥损坏导致 SSH 无法正常工作
**修复密钥目录权限**
chmod 755 /etc/ssh
**修复私钥权限（必须600）**
chmod 600 /etc/ssh/ssh_host_*_key
**修复公钥权限（必须644）**
chmod 644 /etc/ssh/ssh_host_*_key.pub
**修复所属用户组**
chown root:root /etc/ssh/ssh_host_*
**备份旧密钥**
mkdir -p /etc/ssh/old_keys
mv /etc/ssh/ssh_host_* /etc/ssh/old_keys/
**自动生成新密钥并重启SSH**
dpkg-reconfigure openssh-server
systemctl restart ssh.service
**测试本地端口连通性**
nc -zv 127.0.0.1 your_port

## 关闭ipv6

vim /etc/sysctl.conf
在文件末尾添加以下 3 行：

```bash
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

保存退出后，执行命令使配置生效
`sysctl -p`

## docker install

**官方文档**
<https://docs.docker.com/compose/install/linux/>

1. 卸载可能冲突的docker

    ```bash
    sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)
    ```

2. Add Docker's official GPG key:

    ```bash
    sudo apt update
    sudo apt install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    ```

3. Add the repository to Apt sources:

   ```bash
    sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
    Types: deb
    URIs: https://download.docker.com/linux/ubuntu
    Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
    Components: stable
    Signed-By: /etc/apt/keyrings/docker.asc
    EOF
    ```

    ```bash
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    sudo systemctl status docker
    ```

4. Uninstall docker engine

    ```bash
    sudo apt purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras

    sudo rm -rf /var/lib/docker
    sudo rm -rf /var/lib/containerd

    sudo rm /etc/apt/sources.list.d/docker.sources
    sudo rm /etc/apt/keyrings/docker.asc
    ```

## 防火墙ufw相关知识

如果只想开启12345端口的ipv4访问
ufw allow proto tcp to 0.0.0.0/0 port 12345

同理，如果只想开启12345端口的ipv6访问
ufw allow proto tcp to ::/0 port 12345

如果想限定12345端口的来访ip范围
ufw allow from 192.168.1.0/24 to any port 12345

如果想限定12345端口tcp协议的来访ip范围
ufw allow proto tcp from 192.168.1.0/24 to any port 12345

查看ufw当前开放的所有端口、规则
ufw status verbose

编辑ufw配置文件本身来关闭IPv6： sudo nano /etc/default/ufw 将这一行改为：IPV6=yes 改为 IPV6=no 




