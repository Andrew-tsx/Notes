# 流量通信路径分析
我对流量的流转非常感兴趣，想要搞明白在跨国通信过程中流量到底是如何流转的。首先规定以下内容作为分析基准：
硬件层面：
1，使用运营商流量的安卓手机
2，家中路由器，WAN口接入光猫的宽带，另外两个LAN口接其他电脑
3，家中pc（192.168.5.2），家中nas（192.168.5.3）
4，一台位于东京的vps1，具有公网ip（123.1.1.2）
软件层面：
1，wireguard，udp协议，端口为默认
2，v2rayN，有一些已订阅节点，例如新加坡、美国等
3，外部网站，例如谷歌，twitch等

流量流转路径：
1，安卓手机——启动wireguard——vps1——家中nas
2，安卓手机——启动wireguard——vps1——外部网站
3，家中pc——启动v2rayN——外部网站

请告诉我：
1，在三条路径中，数据包是怎么发送和接收的？详细到具体的ip、端口（可拿虚拟的举例），并结合物理层面的光缆给出解释
2，wireguard和v2rayN起到的作用是什么？有什么区别和差异？
3，三条路径中，影响到通信质量的原因有哪些？例如带宽、拦截等

这是你的问题，第一是买ip不知道养，上来就搭梯子。第二是协议百分百有问题，用的是防火墙很容易检测的协议。第三是网络速度和丢包率是可以换协议比如像hy优化的。我现在racknerd，晚高峰轻松youtube 60帧1080p，70一年的套餐，走的还是普通国际信道垃圾线路。纯属是你不会玩



高危端口关闭
入站规则
135, 137, 138, 139, 445, 455, 3389, 22, 23, 4988, 1521, 3306, 1433, 4899
出站规则
135, 137, 138, 139, 445, 455, 3389, 23, 4988, 1521, 1433, 4899

https://vpsdeck.com/categories/vps-reviews/

https://vpsdeck.com/xray-wireguard-warp/

https://www.racknerd.com/

wg genkey | tee /etc/wireguard/pc3_private.key | wg pubkey > /etc/wireguard/pc3_public.key

wg genkey | tee /etc/wireguard/android1_private.key | wg pubkey > /etc/wireguard/android1_public.key

wg genkey | tee /etc/wireguard/iphone_private.key | wg pubkey > /etc/wireguard/iphone_public.key

https://keepassxc.org/docs/KeePassXC_GettingStarted#_downloading_keepassxc











