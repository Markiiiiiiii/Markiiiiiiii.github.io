---
title: Nginx针对Wordpress做动静分离
date: 2020-08-13 12:32:21
tags: 
- nginx
- wordpress
categories:
- 运维
---
## nginx 作高性能反向代理服务器
nginx作为高性能的web服务器常常用于各种的反向代理和负载均衡服务器，但是今天遇到一个小小的问题为wordpress做一个动静分离需求。

首先发现这个wordpress使用的url传参并不是使用的访问某个.php页面提交参数，而是直接将参数传给根域名
```
http://192.168.1.1/?s=xxx
```
如果传统的简单访问某个动态语言编写的页面nginx是非常简单直接写一个类似：
```
location ~ *\.php $ {
         proxy_pass http://server_group;
        }
```
就可以将参数传递给了动态服务器。

但是基于这个wordpress的需求我们需要将这么写
```
 location / {
                index index.html index.htm;
                proxy_pass http://html_server_group;
                    if ($query_string ~* "s=(.*)$"){
                        proxy_pass http://server_group;
                        }
                    }
#这里$query_string是传参字符串，s是wp系统中指定的传参变量，（.*）正则表示接受后面所有的字符串，如果多个变量以此类推。
```

这里我们因为实验环境，使用了ngrok把内网服务器映射到了公网，但是我们的nginx代理服务器在google vps上，这时需要在nginx中使用dns解析内网域名。

首先在conf中写入dns解析服务器
```
resolver 8.8.8.8; #使用google的dns解析域名
```
然后写location /为：
```
location / {
                index index.html index.htm;
                proxy_pass http://html_server_group;
                set $serverin e338a4fb64e2.ngrok.io; #定义serverin变量为映射域名
                    if ($query_string ~* "s=(.*)$"){
                       proxy_pass http://$serverin; 
                        }
                     }
```
测试通过