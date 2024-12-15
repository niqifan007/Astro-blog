---
title: 1pane Linux管理面板安装第三方软件库和设置自动更新
published: 2023-09-17
description: '1pane Linux管理面板安装第三方软件库和设置自动更新'
image: 'https://s2.loli.net/2023/09/22/tDY17Ob5GBaxi9J.png'
tags: [1pane]
category: 'Guides'
draft: false 
lang: ''
---
## 1panel Liunx管理面板安装第三方软件库并设置自动更新教程

### 1、准备：安装好的1panel面板

1panel文档：https://1panel.cn/docs/

默认 `1Panel` 安装在 `/opt/` 路径下，如果不是可按需修改。

关于大陆电磁环境复杂，`github` 网络连接可能有问题，可以自行搜寻解决方式，如 `ghproxy` 等。

项目地址：https://github.com/okxlin/appstore

### 2、使用方法

默认`1Panel`安装在`/opt/`路径下，如果不是按需修改以下。

### 2.1 国内网络

> GitHub加速方式
> 
> > -   (本仓库已添加)自建：[https://github.com/hunshcn/gh-proxy](https://github.com/hunshcn/gh-proxy)
> > -   [https://ghp.ci](https://ghp.ci/)

#### 2.1.1 使用 git 命令获取应用

`1Panel`计划任务类型`Shell 脚本`的计划任务框里，添加并执行以下命令，或者终端运行以下命令，

```shell
git clone -b localApps https://ghp.ci/https://github.com/okxlin/appstore /opt/1panel/resource/apps/local/appstore-localApps

cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

rm -rf /opt/1panel/resource/apps/local/appstore-localApps
```

然后应用商店刷新本地应用即可。

#### 2.1.2 使用压缩包方式获取应用

`1Panel`计划任务类型`Shell 脚本`的计划任务框里，添加并执行以下命令，或者终端运行以下命令，

```shell
wget -P /opt/1panel/resource/apps/local https://ghp.ci/https://github.com/okxlin/appstore/archive/refs/heads/localApps.zip

unzip -o -d /opt/1panel/resource/apps/local/ /opt/1panel/resource/apps/local/localApps.zip

cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

rm -rf /opt/1panel/resource/apps/local/appstore-localApps

rm -rf /opt/1panel/resource/apps/local/localApps.zip
```

然后应用商店刷新本地应用即可。

### 2.2 国际互联网络

#### 2.2.1 使用 git 命令获取应用

`1Panel`计划任务类型`Shell 脚本`的计划任务框里，添加并执行以下命令，或者终端运行以下命令，

```shell
git clone -b localApps https://github.com/okxlin/appstore /opt/1panel/resource/apps/local/appstore-localApps

cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

rm -rf /opt/1panel/resource/apps/local/appstore-localApps
```

然后应用商店刷新本地应用即可。

#### 2.2.2 使用压缩包方式获取应用

`1Panel`计划任务类型`Shell 脚本`的计划任务框里，添加并执行以下命令，或者终端运行以下命令，

```shell
wget -P /opt/1panel/resource/apps/local https://github.com/okxlin/appstore/archive/refs/heads/localApps.zip

unzip -o -d /opt/1panel/resource/apps/local/ /opt/1panel/resource/apps/local/localApps.zip

cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

rm -rf /opt/1panel/resource/apps/local/appstore-localApps

rm -rf /opt/1panel/resource/apps/local/localApps.zip
```

然后应用商店刷新本地应用即可。

