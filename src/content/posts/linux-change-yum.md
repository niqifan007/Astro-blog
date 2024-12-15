---
title: Centos7更换yum源
published: 2023-10-14
description: 'Centos7更换yum源'
image: 'https://esay.579878700.xyz//i/2023/10/15/27z89t.png'
tags: [linux]
category: 'Guides'
draft: false 
lang: ''
---
# Centos7更换yum源

在 CentOS 7 中，默认的 Yum 软件包源可能在国内的网络环境下会有较慢的下载速度。本文旨在提供简明的步骤，指导你如何将默认的 Yum 源更换为阿里云的 Yum 源，从而获得更加流畅的软件包管理体验。

## 背景

Yum 源是 CentOS 等基于 Red Hat 的发行版用于管理与更新软件包的主要工具。由于网络等多种原因，国内访问国外 Yum 源的速度可能较慢，因此，更换为国内的镜像源，如阿里云提供的 Yum 源，通常能够得到更好的速度和稳定性。

## 详细操作步骤

### 步骤一：下载wget

如果安装的Centos为最小版本，则系统中未安装[wget命令](https://so.csdn.net/so/search?q=wget命令&spm=1001.2101.3001.7020)，因此需在更换yum源之前先将wget下载。

```bash
yum -y install wget
```

### 步骤二：备份现有 Yum 源

在开始操作之前，建议备份现有的 Yum 源配置，以防不测。

#### 2.1 进入yum源目录下

```
[root@bd01 ~]# cd /etc/yum.repos.d/
```

#### 2.2 查看当前目录下的内容

```
[root@bd01 yum.repos.d]# ls -l
total 40
-rw-r--r--. 1 root root 1572 Dec  1  2016 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Nov 23  2020 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Nov 23  2020 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Nov 23  2020 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Nov 23  2020 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Nov 23  2020 CentOS-Sources.repo
-rw-r--r--. 1 root root 8515 Nov 23  2020 CentOS-Vault.repo
-rw-r--r--. 1 root root  616 Nov 23  2020 CentOS-x86_64-kernel.repo
```

#### 2.3 重命名yum源

```
[root@bd01 yum.repos.d]# mv CentOS-Base.repo CentOS-Base.repo.bak
[root@bd01 yum.repos.d]# ls -l
total 40
-rw-r--r--. 1 root root 1572 Dec  1  2016 CentOS-Base.repo.bak
-rw-r--r--. 1 root root 1309 Nov 23  2020 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Nov 23  2020 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Nov 23  2020 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Nov 23  2020 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Nov 23  2020 CentOS-Sources.repo
-rw-r--r--. 1 root root 8515 Nov 23  2020 CentOS-Vault.repo
-rw-r--r--. 1 root root  616 Nov 23  2020 CentOS-x86_64-kernel.repo
```

### 步骤三：下载yum源

阿里-centos7：http://mirrors.aliyun.com/repo/Centos-7.repo
网易-centos7：http://mirrors.163.com/.help/CentOS7-Base-163.repo
这里提供两种yum源的下载地址，本下载需要虚拟机联网。
案例采用阿里的yum源，网易的操作步骤类同。

``` bash
cd /etc/yum.repos.d/
rm -f CentOS-Base.repo  # 删除现有的 Yum 源配置文件
wget http://mirrors.aliyun.com/repo/Centos-7.repo
```

### 步骤四：修改yum源名称

将下载的yum源的名修改为系统默认的yum源名称。

```
mv Centos-7.repo CentOS-Base.repo
```

### 步骤五：清理 Yum 缓存

更换 Yum 源后，我们需要清理 Yum 的缓存以确保下次安装或更新软件包时使用的是新源。

```bash
yum clean all
yum makecache
```

### 步骤六：验证 Yum 源是否已更换成功

我们可以尝试使用以下命令来更新系统，或者安装一个软件包以验证新的 Yum 源是否工作正常。

``` bash
yum -y update
```

或者尝试安装一个软件包：

``` bash
yum -y install vim
```

如果以上命令能够正常工作，说明 Yum 源已经成功更换为阿里云源。

## 结语

更换为阿里云的 Yum 源能够使我们在使用 CentOS 7 时获得更快的软件包下载速度，提高系统更新与维护的效率。希望本教程能够帮助到正在寻找如何更换 Yum 源的朋友们！

如遇到任何问题或疑问，欢迎留言交流。