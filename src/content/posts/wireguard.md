---
title: Wireguard 搭建异地组网
published: 2024-06-13
description: 'Wireguard 搭建异地组网'
image: 'https://esay.579878700.xyz//i/2024/06/13/sqbu7m.jpg'
tags: [wireguard]
category: 'Guides'
draft: false 
lang: ''
---
### 前言

之前就一直听说 Wireguard 可以进行异地组网，但是多次尝试发现只能达到点对点通信的效果，需要在每台机器上搭建客户端显然是不合理的。并且由于搭建了 tailscale，并且是基于 wireguard 的，所以就没再折腾了。

最近因为群友提到这个，所以又折腾了一下，并且互相交流了一番，发现果然可以实现异地组网。但是，组网却比 tailscale 麻烦上许多，并且有些出现的问题可能很难发现。但是基于本人多次尝试，发现了一个对自己来说比较方便的搭建方式。

### 服务器搭建wg-easy

#### 1\. 安装wireguard内核

由于使用的是 centos7 服务器，默认内核不支持 wireguard。

如果不进行安装可能出现如下日志：

> Mon May 27 03:19:09 UTC 2024: Starting Wireguard /etc/wireguard/wg0.conf  
> \[#\] ip link add wg0 type wireguard  
> RTNETLINK answers: Not supported  
> Unable to access interface: Protocol not supported  
> \[#\] ip link delete dev wg0  
> Cannot find device "wg0"  
> Adding iptables NAT rule

运行如下命令安装即可：

```sh
yum update && \ yum install epel-release elrepo-release -y && \ yum install yum-plugin-elrepo -y && \ yum install kmod-wireguard wireguard-tools -y
```

#### 2\. 加载模块

如果不进行如下命令操作，可能后续会出现能 ping 通，但是不能够进行 ssh 或者 http 访问的情况。

```sh
sudo modprobe iptable_filter && \ sudo modprobe ip6table_filter
```

#### 3\. 安装wg-easy

wg-easy 是一个可以快速搭建 wireguard 服务端的项目，提供了一个方便的 ui 界面，生成客户端只需要在界面生成，然后即可下载配置文件。

当然了，之前因为没有搭建成功也是由于它的弊端，因为他的配置文件不能手动修改，**所以就导致客户端配置了路由，但是服务器不知道往哪里发的问题**。

所以，搭建 wg-easy 只是为了过度。

```sh
version: '3.1' services: wg-easy: image: weejewel/wg-easy:nightly container_name: wg-easy environment: - WG_HOST=<Server IP> - PASSWORD=<PASSWORD> - WG_ALLOWED_IPS=10.8.0.0/24 - WG_PERSISTENT_KEEPALIVE=25 - WG_DEFAULT_DNS=114.114.114.114 - UI_TRAFFIC_STATS=true volumes: - /home/docker/wireguard:/etc/wireguard network_mode: "host" # 51820 为 wireguard 数据传输的端口，51821 为 wg-easy 端口 # ports: # - 51820:51820/udp # - 51821:51821/tcp cap_add: - NET_ADMIN - SYS_MODULE privileged: true restart: unless-stopped
```

这里需要注意的配置有三个：

-   WG\_HOST：填写服务器 IP
-   PASSWORD：访问 wg-easy 的密码
-   WG\_ALLOWED\_IPS：客户端路由地址，即客户端下载配置文件都会携带该配置

### 安装wireguard客户端

#### 创建客户端配置文件

访问 `ip:51821`，然后输入密码即可进入 wg-easy 图形界面，如下图所示。（如果需要配置反代可以自行配置）

![20240527153617.png](https://img.amjun.com/uploads/2024/05/665437f709cbc.png)

wireguard 官方客户端下载地址：[https://www.wireguard.com/install/#ubuntu-module-tools](https://www.nodeseek.com/jump?to=https%3A%2F%2Fwww.wireguard.com%2Finstall%2F%23ubuntu-module-tools)

#### windows安装

在网页端点击下载，然后导入配置文件即可。

![20240527155514.png](https://img.amjun.com/uploads/2024/05/66543c67c40c0.png)

#### 移动端安装

直接扫码即可导入配置。

![20240527155651.png](https://img.amjun.com/uploads/2024/05/66543cc9892b2.png)

#### linux安装

在 linux 上，个人喜欢使用 docker 进行安装。当然，如果使用的依然是 centos，那么还需要像上面那样**安装 wireguard 内核**。

下载配置文件，将其修改为 `wg0.conf`，然后放在 `/home/docker/wireguard` 目录下。

再运行如下命令：

```sh
docker run -d \ --privileged \ --net=host \ --restart=always \ --name wireguard-client \ --cap-add NET_ADMIN \ --cap-add SYS_MODULE \ -v /home/docker/wireguard/wg0.conf:/etc/wireguard/wg0.conf \ --restart unless-stopped \ cmulk/wireguard-docker:alpine
```

### 异地组网

如果按上面的步骤，应该可以实现点对点进行通信，即连接上的客户端会像这样出现红点。

![20240527153617.png](https://img.amjun.com/uploads/2024/05/665437f709cbc.png)

比如网络情况如下，并且需要移动端能够实现移动端直接输入服务器 A 和服务器 B 的内网 ip 即可访问服务。

![20240527162535.png](https://img.amjun.com/uploads/2024/05/6654438520010.png)

要实现上述目的，要进行如下操作。

#### 1\. 保证移动端路由

![20240527155651.png](https://img.amjun.com/uploads/2024/05/66543cc9892b2.png)

这里显示的配置意思是，wireguard 会代理 `10.8.0.0/24, 172.21.9.0/24, 172.30.1.0/24, 172.26.1.0/24, 172.20.2.0/23` 的流量，将所有匹配的流量都发往服务器。

#### 2\. 修改服务器配置

例如我的内网设备为 `gw` 这台设备，当服务器收到 `172.21.9.0/24, 172.30.1.0/24, 172.26.1.0/24, 172.20.2.0/23` 请求时，需要将其转发到 `gw` 设备上。

修改对应的 `AllowedIPs` 即可。

![20240527163153.png](https://img.amjun.com/uploads/2024/05/665444ff695d9.png)

删除 `wg-easy` 容器。

> wg-easy 的作用在这里就结束了，它的作用就是为了方便地生成客户端地配置，现在客户端基本已经配置好了，并且需要修改 `wg0.conf`，而它会覆盖配置，所以需要删除。

```sh
docker rm -f wg-easy
```

运行如下命令：

```sh
docker run -d \ --privileged \ --net=host \ --restart=always \ --name wireguard-client \ --cap-add NET_ADMIN \ --cap-add SYS_MODULE \ -v /home/docker/wireguard/wg0.conf:/etc/wireguard/wg0.conf \ --restart unless-stopped \ cmulk/wireguard-docker:alpine
```

#### 3\. 修改客户端配置

**如果内网设备为路由器，不需要进行该操作。**

首先内网设备需要开启 ipv4 转发，运行 `cat /proc/sys/net/ipv4/ip_forward` 如果结果为 1，表示转发已开启。

否则需要修改 `/etc/sysctl.conf` 文件，将 `#net.ipv4.ip_forward=1` 修改为 `net.ipv4.ip_forward=1`。再执行 `sysctl -p`。

另外，`wg0.conf` 需要添加如下配置：

```sh
PostUp = iptables -A FORWARD -i client_route -j ACCEPT; iptables -A FORWARD -o client_route -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; iptables -A FORWARD -i eth0 -j ACCEPT; iptables -A FORWARD -o eth0 -j ACCEPT; iptables -t nat -A POSTROUTING -o client_route -j MASQUERADE PostDown = iptables -D FORWARD -i client_route -j ACCEPT; iptables -D FORWARD -o client_route -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; iptables -D FORWARD -i eth0 -j ACCEPT; iptables -D FORWARD -o eth0 -j ACCEPT; iptables -t nat -D POSTROUTING -o client_route -j MASQUERADE
```

![20240527163637.png](https://img.amjun.com/uploads/2024/05/6654461a5942c.png)

如果不添加上面的配置，将服务路由到其他服务器，因为没有进行源地址 nat 转换，即使数据发送到了服务器 A/B 上，数据也回不来，因为它不知道 `10.8.0.0/24` 的路由。照上面的配置，在发送到其他服务器时，会将源地址转换成内网设备的地址。