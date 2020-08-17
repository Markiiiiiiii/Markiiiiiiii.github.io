---
title: Ubuntu中配置Mail命令使用外部STMP发送邮件
date: 2020-08-17 12:15:43
tags:
- Ubuntu
- Linux
categories:
- 运维
---
# Ubuntu中配置mail命令使用外部的stmp发送邮件
因为在日常运维过程中我们需要使用mail命令，实时的发送服务器的一些状态参数或者监控服务器的运行状态，所以在mail的日常应用中还是非常的高频，鉴于我们做的站点大部分不开启自己本身的邮箱服务所以我们还是尽量使用外部的stmp邮箱服务来完成我们这个发信过程。

！！！注意！！！Ubuntu于其他发行版的不同，需要注意安装mail命令和相关软件包时的几个坑

### 1.安装heirloom-mailx mailutils
需要将
```
deb http://cz.archive.ubuntu.com/ubuntu xenial main universe
```
添加到/etc/apt/sources.list中并执行apt-get update后再apt install heirloom-mailx mailutils

因为heirloom-mailx并不在官方的源内。

### 2.修改配置文件 /etc/s-nail.rc
添加如下配置
```
set from=xxx@xxx.com                  #设置发送邮箱
set smtp=smtps://smtp.xxx.com         #设置smtp服务器和端口
set smtp-auth-user=cm@xxx.com         #设置用户名，记得加域名啊
set smtp-auth-password=xxxxx          #邮箱授权码
set smtp-auth=xxx@xxx.com             #用户名
```

### 3.修改服务器的主机名称
```
$> hostname
xxxxx
```
一般VPS的名称大部分为一个无序字符串组成，这个字符串在发信时会不符合目的邮件服务器的要求一般会出现550错误，126会出现以下的：
```
<<< 550 MI:IMF 126 mx2,IMmowACXz0un8TlfJMPIAg--.47921S3 1597632936 http://mail.163.com/help/help_spam_16.htm?ip=173.82.56.165&hostid=mx2&time=1597632936
```
google也是会出现类似的550错误，这个错误是因为我们的主机名不符合邮件服务器的要求，将你的主机名修改xxx.com类似的形式即可发信成功

```
hostname xxx.xxx.com
#临时修改主机名，重启后恢复原字符串主机名
```
或者可以
```
vim /etc/hostname
#永久修改为xxx.xxx.com 注意需reboot重启。
```

### 4.注意尽量使用邮箱给的授权码，不要使用邮箱的登录码
