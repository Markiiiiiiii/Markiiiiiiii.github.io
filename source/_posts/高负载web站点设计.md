---
title: '高负载web站点设计 (一)'
date: 2020-08-05 14:08:38
tags:
- Ubuntu
- nginx
categories:
- 运维
---

## A.负载平衡服务器

- A.1. ngnix反向代理服务根据B服务组的最短时间或负载来自动均衡。

## B.静态页面服务器集群

### b.1. nginx WEB业务集群

- 使用rsync服务同步C.1.服务器生成的静态页面和相关的静态资源。（扩展思路：将GIF，JPG等静态资源单独使用dfs服务器分流）

## C.内网动态服务器

### c.1.  Wordpress内容生产服务器

- wordpress服务器使用wp2static插件生成静态页面，wp2static插件可以识别之前生成的页面从而减少服务器负载。

- wordpress服务器使用inonifywait脚本监控wp2static插件生成的静态页面目录，如果产生文件的变更（创建、删除、修改、移动）和目录结构的变更后，则通过rsync服务器同步远程服务器。

### c.2.   Pyspider爬虫服务器

- Pyspider爬虫框架爬取需要的数据
- 安装python-wordpress-xmlrpc库， 使用wordpress的rpc服务接受Pyspider爬取经过清洗后的数据发布到wordpress服务器。