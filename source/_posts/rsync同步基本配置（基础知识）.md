---
title: 'rsync同步基本配置（基础知识）'
date: 2020-08-03 10:04:38
tags: 
- Ubuntu
- rsync
categories:
- 运维
---

## rrsync同步基本配置（基础知识）
### Ubuntu 20.x 环境
### 服务器端：
```
	[123]
  comment = 123 static html files server
	path = /var/www/html/site
	use chroot = no
	max connections=20
	lock file = /var/lock/rsyncd
	read only = yes
	list = yes
	uid = 0
	gid = 0
      auth users = 123clint
      secrets file = /etc/rsyncd.secrets
      strict modes = yes
      port=873
	ignore errors = yes
	ignore nonreadable = yes
	transfer logging = yes
	timeout = 600
	refuse options = checksum dry-run
	dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz
```
增加/etc/rsyncd.secrets文件
```
123clint：密码123456
```
启动服务器：
```
/etc/init.d/rsync --daemon
```
### 客户端：
增加/etc/rsyncd.secrets文件
```
密码123456
```

客户端连接服务器命令
```
rsync -vzrtopg --delete --progress 123clint@服务器地址::123 /var/www/staichtml（客户端备份目录位置） --password-file=/etc/rsyncd.secrets
```
 **注意**
客户端和服务器端的rsyncd.secrets 文件均需要chmod 600 权限
```
chmod 600 rsyncd.secrets 
```
