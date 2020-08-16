---
title: Google虚拟机vps的本地化SSH登陆设置
date: 2020-08-11 08:50:51
tags:
- Google
- VPS
categories:
- 运维
---
## Google cloud VPS 本地使用SSH登陆
昨天发现google cloud上发现还有2K+的余额，正好最近在学习nginx的负载平衡，拿来做个vps来做nginx的服务器来实例化一下。vps做好后结果发现本地ssh登陆是要做一些小小的设置。
```
ssh-keygen -f googlekey   
#创建一个googlekey公私钥
cat googlekey.pub
#输出公钥
```
打开google cloud的元数据--ssh--修改--添加一项

将公钥输入保存

本地终端中输入：
```
ssh -i googlekey username@ip
#username这时是在元数据中显示的name
```