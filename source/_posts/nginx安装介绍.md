---
title: nginx安装介绍
date: 2019-07-27 22:00:30
tags:
	- nginx
categories: Nginx
toc: true
---

nginx的安装介绍。

<!-- more -->

<h3> 一、环境调试确认 </h3>

1.确认系统网络

```shell
ping www.baidu.com
```

检查能否ping通

2.确认yum可用

```shell
yum list|grep gcc
```

3.确认关闭iptables规则

```shell
Iptables -L    查看是否有iptables规则
Iptables -F    关闭iptables规则
Iptables -t nat -L    再次查看是否有iptables规则
```

4.确认停用selinux

```shell
Getenforce     查看selinux是否开启
Setenforce  0  关闭selinux（只是临时关闭
```

> 修改/etc/sysconfig/selinux文件可以永久地禁用它。

![](关闭selinu.png)

5.安装基础依赖

> gcc和gcc-c++以及相应依赖

```shell
yum -y install gcc gcc-c++ autoconf prce prce-devel make automake
```

> 安装vim

```shell
yum -y install wget httpd-tools vim
cd /opt;mkdir app download logs work backup
```

![](nginx01.png)

<h3> 二、下载安装 </h3>

前往nginx官方下载页面：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

![](nginx02.png)

> 对应版本介绍

- Mainline version-开发版
- Stable version-稳定版
- Legacy version-历史版本

点击页面下方的

> Linux packages for [stable and mainline](http://nginx.org/en/linux_packages.html) versions. 进入对应不同Linux系统的安装介绍

本系统是centos7，所以根据提示：

> create the file named /etc/yum.repos.d/nginx.repo with the following contents:

![](nginx03.png)

查看nginx yum源

![](nginx04png.png)

```shell
yum install nginx
```

![](nginx05.png)

安装完成，查看安装好的nginx版本：

![](nginx06.png)

