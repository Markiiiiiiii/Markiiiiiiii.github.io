---
title: rsync+inofity内网向外网同步数据(一)
date: 2020-08-03 11:22:53
tags:
- Ubuntu
- rsync
- inotify
categories:
- 运维
---
## 设计要求 
- 内网web服务器生成html页面
- 内网向外网同步生成的html页面和目录
- 外网服务器纯静态资源访问

## 系统环境
- 内网 Ubuntu 20.x + Ngnix + PHP + Mysql + Wordpress
- 外网 Ubuntu 20.x + Ngnix 

### 外网服务器配置
外网服务器必须提供rsync --daemon服务以监听内网服务器发出的同步指令
```
[123]
  comment = 123 static html files server
	path = /var/www/staichtml/
	use chroot = no
	max connections=20
	lock file = /var/lock/rsyncd
	read only = no
  write only = no
	list = yes
	uid = rsync
	gid = rsync
  auth users = rsync_clint
  secrets file = /etc/rsyncd.secrets
	strict modes = yes
  port=873
  pid file = /var/run/rsyncd.pid
  log file = /var/log/rsyncd.log
  log format = %t %a %m %f %b
	ignore errors = yes
	ignore nonreadable = yes
	transfer logging = yes
	timeout = 600
	refuse options = checksum dry-run
	dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz

```
创建 /etc/rsyncd.secrets 并 chmod 600 权限
```
rsync_clint:密码123
```
添加一个系统用户为rsync的用户 *该用户不具备登录权限*

    ```
    useradd -s /sbin/nologin -M rsync
    ```
**！！！注意！！！必须 chmod 777 /var/www/staichtml/ 在实际操作中发现没有x权限rsync服务无法写入文件**

### 内网服务器配置
*略过相关web服务配置*
创建 /etc/rsyncd.secrets 并 chmod 600 权限，仅存放密码使用
```
密码123
```
安装apt install inofity-tool

编写简单实现的.sh文件并 chmod +x xxxx.sh
```
##测试代码：
#!/bin/bash
src=/var/www/html/mystaticsite/ #同步目录路径
ip=111.111.111.111

/usr/bin/inotifywait -rm  --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib,move $src | while read file
   do
     rsync -avz --delete --progress $src rsync_clint@$ip::123 --password-file=/etc/rsyncd.secrets
    echo "${file} was rsynced" >> /tmp/rsync.log 2>&1
   done
```
```
！！！注意！！！
必须将$src的位置写在连接之前不能放在连接之后，不然外网服务器无法接受到文件。
这一点与网上很多教程向左，卡了我两天多的时间才发现的:(
```

### 生产环境代码：
```
#!/bin/bash

src=/var/www/html/ #同步目录路径
user=rsync_clint
ip1=7xx.xxx.xxx.xxx
ip2=1xx.xx.xxx.xx
des=123
pswfile=/etc/rsyncd.secrets

cd ${src}
/usr/bin/inotifywait -rmq  --format '%Xe %w%f' -e modify,delete,create,attrib,close_write,move ./ | while read file
   do
      INO_EVENT=$(echo $file | awk'{print $1}')
      INO_FILE=$(echo $file | awk'{print $2}')
      echo "----------------${data}-----------------"
      echo $file
      
      if [[$INO_EVENT =~ 'CREATE']] || [[$INO_EVENT =~ 'MODIFY']] || [[$INO_EVENT =~ 'CLOSE_WRITE']] || [[$INO_EVENT =~ 'MOVE_TO']]
      then
          echo 'Creat or Modify or Close_write or Moved_to'
          rsync -avzcR --password-file=${pswfile} $(dirname ${INO_FILE}) ${user}@$ip{1}::${des} &&
          rsync -avzcR --password-file=${pswfile} $(dirname ${INO_FILE}) ${user}@$ip{2}::${des} 
      fi

      if [[$INO_EVENT =~ 'DELETE']] || [[$INO_EVENT =~ 'MOVE_FROM']]
      then
          echo 'Delete or Moved_from'
          rsync -avzR --delete --password-file=${pswfile} $(dirname ${INO_FILE}) ${user}@$ip{1}::${des} &&
          rsync -avzR --delete --password-file=${pswfile} $(dirname ${INO_FILE}) ${user}@$ip{2}::${des}  
      fi

      if [[$INO_EVENT =~ 'ATTRIB']]
      then
          echo 'Attrib'
          if [ !-d "$INO_FILE" ]
          then
            rsync -avzcR  --password-file=${pswfile} $(dirname ${INO_FILE}) ${user}@$ip{1}::${des} &&
            rsync -avzcR  --password-file=${pswfile} $(dirname ${INO_FILE}) ${user}@$ip{2}::${des}
          fi
      fi

   done
```
因为inotify不会监控到为启动时的变化，所以直接在crontab里增加一个2小时全量目录同步
```
crontab -e
* */2 * * * rsync -avz --password-file=/etc/rsyncd.secrets/var/www/html/  rsync_clint@1xxx.xxx.xxx.xxx::123 && rsync -avz --password-file=/etc/rsyncd.secrets /var/www/html/ rrsync_clint@7xx.xxx.xxx.xx::123

```