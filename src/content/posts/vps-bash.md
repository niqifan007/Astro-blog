---
title: 常用VPS脚本合集
published: 2025-01-11
description: '常用VPS脚本合集'
image: 'https://i.111666.best/image/N1i9aiGyJe2WLIvMi07zta.jpg'
tags: [Guides]
category: 'linux'
draft: false 
lang: ''
---
## 1、DD重装脚本

史上最强脚本

```bash
wget --no-check-certificate -qO InstallNET.sh 'https://raw.githubusercontent.com/leitbogioro/Tools/master/Linux_reinstall/InstallNET.sh' && chmod a+x InstallNET.sh && bash InstallNET.sh -debian 12 -pwd 'password'
```

萌咖大佬的脚本

```perl
bash <(wget --no-check-certificate -qO- 'https://raw.githubusercontent.com/MoeClub/Note/master/InstallNET.sh') -d 11 -v 64 -p 密码 -port 端口 -a -firmware
```

beta.gs大佬的脚本

```perl
wget --no-check-certificate -O NewReinstall.sh https://raw.githubusercontent.com/fcurrk/reinstall/master/NewReinstall.sh && chmod a+x NewReinstall.sh && bash NewReinstall.sh
```

DD windows（使用史上最强DD脚本）

```ruby
bash <(curl -sSL https://raw.githubusercontent.com/leitbogioro/Tools/master/Linux_reinstall/InstallNET.sh) -windows 10 -lang "cn"
```

```undefined
账户：Administrator 密码：Teddysun.com
```

## 2、综合测试脚本

LemonBench

```bash
wget -qO- https://raw.githubusercontent.com/LemonBench/LemonBench/main/LemonBench.sh | bash -s -- --fast
```

融合怪

```perl
bash <(wget -qO- --no-check-certificate https://gitlab.com/spiritysdx/za/-/raw/main/ecs.sh)
```

NodeBench

```bash
bash <(curl -sL https://raw.githubusercontent.com/LloydAsp/NodeBench/main/NodeBench.sh)
```

## 3、性能测试

GB6 跑分脚本，附带宽测试：

```undefined
curl -sL yabs.sh | bash
```

GB6 剔除带宽测试，因为都是国外节点测试，国内跑没多大意义：

```lua
curl -sL yabs.sh | bash -s -- -i
```

GB5 跑分脚本，附带宽测试：

```undefined
curl -sL yabs.sh | bash -5
```

GB5 剔除带宽测试：

```lua
curl -sL yabs.sh | bash -s -- -i -5
```

## 4、流媒体及IP质量测试

最常用版本

```scss
bash <(curl -L -s check.unlock.media)
```

原生检测脚本

```scss
bash <(curl -sL Media.Check.Place)
```

准确度最高

```bash
bash <(curl -L -s https://github.com/1-stream/RegionRestrictionCheck/raw/main/check.sh)
```

IP质量体检脚本

```scss
bash <(curl -sL IP.Check.Place)
```

## 5、测速脚本

Speedtest

```bash
bash <(curl -sL bash.icu/speedtest)
```

Taier

```bash
bash <(curl -sL res.yserver.ink/taier.sh)
```

hyperspeed

```bash
bash <(curl -Lso- https://bench.im/hyperspeed)
```

全球测速

```undefined
curl -sL network-speed.xyz | bash
```

## 6、回程测试

直接显示回程（小白用这个）

```bash
curl https://raw.githubusercontent.com/ludashi2020/backtrace/main/install.sh -sSf | sh
```

回程详细测试（推荐）

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/Chennhaoo/Shell_Bash/master/AutoTrace.sh && chmod +x AutoTrace.sh && bash AutoTrace.sh
```

```ruby
wget https://ghproxy.com/https://raw.githubusercontent.com/vpsxb/testrace/main/testrace.sh -O testrace.sh && bash testrace.sh
```

## 7、功能脚本

添加SWAP

```ruby
wget https://www.moerats.com/usr/shell/swap.sh && bash swap.sh
```

Fail2ban

```bash
wget --no-check-certificate https://raw.githubusercontent.com/FunctionClub/Fail2ban/master/fail2ban.sh && bash fail2ban.sh 2>&1 | tee fail2ban.log
```

一键开启BBR，适用于较新的Debian、Ubuntu

```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf sysctl -p sysctl net.ipv4.tcp_available_congestion_control lsmod | grep bbr
```

多功能BBR安装脚本

```bash
wget -N --no-check-certificate "https://gist.github.com/zeruns/a0ec603f20d1b86de6a774a8ba27588f/raw/4f9957ae23f5efb2bb7c57a198ae2cffebfb1c56/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

锐速/BBRPLUS/BBR2/BBR3

```bash
wget -O tcpx.sh "https://github.com/ylx2016/Linux-NetSpeed/raw/master/tcpx.sh" && chmod +x tcpx.sh && ./tcpx.sh
```

TCP窗口调优

```bash
wget http://sh.nekoneko.cloud/tools.sh -O tools.sh && bash tools.sh
```

添加warp

```perl
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh [option] [lisence/url/token]
```

25端口开放测试

```undefined
telnet smtp.aol.com 25
```

## 8、一键安装常用环境及软件

docker

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

Python

```bash
curl -O https://raw.githubusercontent.com/lx969788249/lxspacepy/master/pyinstall.sh && chmod +x pyinstall.sh && ./pyinstall.sh
```

iperf3

```undefined
apt install iperf3
```

realm

```bash
bash <(curl -L https://raw.githubusercontent.com/zhouh047/realm-oneclick-install/main/realm.sh) -i
```

gost

```bash
wget --no-check-certificate -O gost.sh https://raw.githubusercontent.com/qqrrooty/EZgost/main/gost.sh && chmod +x gost.sh && ./gost.sh
```

极光面板

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/Aurora-Admin-Panel/deploy/main/install.sh)
```

哪吒监控

```bash
curl -L https://raw.githubusercontent.com/naiba/nezha/master/script/install.sh -o nezha.sh && chmod +x nezha.sh && sudo ./nezha.sh
```

WARP

```perl
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh
```

Aria2

```bash
wget -N git.io/aria2.sh && chmod +x aria2.sh && ./aria2.sh
```

宝塔

```bash
wget -O install.sh http://v7.hostcli.com/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

PVE虚拟化

```ruby
bash <(wget -qO- --no-check-certificate https://raw.githubusercontent.com/oneclickvirt/pve/main/scripts/build_backend.sh)
```

Argox

```bash
bash <(wget -qO- https://raw.githubusercontent.com/fscarmen/argox/main/argox.sh)
```

1panel

```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh
```

## 9、综合功能脚本

科技lion

```scss
apt update -y && apt install -y curl bash <(curl -sL kejilion.sh)
```

SKY-BOX

```bash
wget -O box.sh https://raw.githubusercontent.com/BlueSkyXN/SKY-BOX/main/box.sh && chmod +x box.sh && clear && ./box.sh
```