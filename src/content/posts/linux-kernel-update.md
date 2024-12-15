---
title: 在 CentOS 7 上更新到最新 LTS 内核
published: 2023-10-14
description: '在 CentOS 7 上更新到最新 LTS 内核教程'
image: 'https://esay.579878700.xyz//i/2023/10/15/29rkoy.png'
tags: [linux]
category: 'Guides'
draft: false 
lang: ''
---
# 在 CentOS 7 上更新到最新 LTS 内核

本篇博客将介绍如何完成在 CentOS 7 系统中更换 Yum 源为阿里云源，并利用这个源更新到最新的长期支持（LTS）内核的过程。

仅查看版本信息

```bash
uname -r
```

![img](https://esay.579878700.xyz//i/2023/10/15/vqhwu.png)

查看版本信息及相关内容

```bash
uname -a
```

![img](https://esay.579878700.xyz//i/2023/10/15/vsgfv.png)

 通过绝对路径查看查看版本信息及相关内容

```bash
cat /proc/version
```

![img](https://esay.579878700.xyz//i/2023/10/15/vt9wm.png)

 通过绝对路径查看查看版本信息

```bash
cat /etc/redhat-release
```

![img](https://esay.579878700.xyz//i/2023/10/15/vu42x.png)

## 1.安装最新的 LTS 内核

更新版本使用yum,建议提前更新yum源仓库

**更新yum源仓库**

```
yum -y update
```

![](https://esay.579878700.xyz//i/2023/10/15/vvgjs.png)

更新完成后，启用 ELRepo 仓库并安装ELRepo仓库的yum源

 ELRepo 仓库是基于社区的用于企业级 Linux 仓库，提供对 RedHat Enterprise (RHEL) 和 其他基于 RHEL的 Linux 发行版（CentOS、Scientific、Fedora 等）的支持。

#### 1、导入ELRepo仓库的公共密钥

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

![](https://esay.579878700.xyz//i/2023/10/15/1w346e.png)

 2、安装ELRepo仓库的yum源

```
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```

![](https://esay.579878700.xyz//i/2023/10/15/1w44s1.png)

#### 3、查询可用内核版本

```
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
```

![](https://esay.579878700.xyz//i/2023/10/15/1w5b15.png)

  可选版本如下：yum --disablerepo="\*" --enablerepo="elrepo-kernel" list available

![](https://esay.579878700.xyz//i/2023/10/15/1xnsdo.png)

其中lt表示long-term，即主线版本，该版本建议慎重选择。

其中mt表示latest mainline，即长期稳定版本，稳定可靠，建议安装该版本

如：yum -y --enablerepo=elrepo-kernel install kernel-lt

#### 4、安装最新的稳定版本内核

```
yum -y --enablerepo=elrepo-kernel install kernel-lt
```

会默认安装lt的最新版本，即5.4.207-1.el7.elrepo，也可根据具体需要指定对应版本。

![](https://esay.579878700.xyz//i/2023/10/15/1wh87d.png)

### 如果国内下载的速度不行，可以试试阿里的源

#### 5.1. 添加 ELRepo 的阿里云源

```bash
nano /etc/yum.repos.d/elrepo.aliyun.repo
```

添加以下内容：

```plaintext
[elrepo]
name=ELRepo.org Community Enterprise Linux Repository - el7
baseurl=https://mirrors.aliyun.com/elrepo/elrepo/el7/$basearch/
        https://mirrors.tuna.tsinghua.edu.cn/elrepo/el7/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

保存并退出。

#### 5.2. 安装最新 LTS 内核

我们可以直接从阿里云的 ELRepo 源安装最新的 LTS 内核：

```bash
yum --enablerepo=elrepo install kernel-lt
```

### 2. 配置 Grub

当下载完成的时候，我们需要告知系统使用新安装的内核版本。首先，编辑 Grub 配置文件：

```bash
nano /etc/default/grub
```

设置 `GRUB_DEFAULT=0`，确保使用新内核启动系统。

```plaintext
GRUB_DEFAULT=0
```

（注意，上面的那个是数字0，不是英文字母o）

接着，重新生成 Grub 配置：

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 3.重启并验证

最后，我们需要重启系统以使用新安装的 LTS 内核。

```bash
reboot
```

重启完成后，使用以下命令验证是否成功使用新的内核版本：

```bash
uname -r
```

#### 删除旧内核（**可选**）

查看系统中的全部内核

```perl
rpm -qa | grep kernel
```

可选择删除3.10版本的内核

```csharp
yum remove kernel-版本
```

## 结语

至此，我们已经成功在 CentOS 7 上更新到最新的 LTS 内核。这个过程旨在提供一个更快速、更平滑的更新和安装过程，希望能帮到你。