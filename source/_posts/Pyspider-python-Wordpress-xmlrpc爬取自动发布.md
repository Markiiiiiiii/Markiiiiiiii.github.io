---
title: Pyspider+python-Wordpress-xmlrpc爬取自动发布
date: 2020-08-08 11:06:40
tags:
- pyspider
- wordpress
categories:
- 编程
---
## Pyspider+python-wordpress-xmlrpc 爬取站点自动发布
Pyspider内代码：
```
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2020-08-07 01:19:41
# Project: xxxxxx

from pyspider.libs.base_handler import *
import re
DIR_PATH = '/var/www/html/mystaticsite/downimages/'

#coding:utf-8
from wordpress_xmlrpc import Client, WordPressPost
from wordpress_xmlrpc.methods.posts import GetPosts, NewPost
from wordpress_xmlrpc.methods.users import GetUserInfo
from wordpress_xmlrpc.methods import posts
from wordpress_xmlrpc.methods import taxonomies
from wordpress_xmlrpc import WordPressTerm
from wordpress_xmlrpc.compat import xmlrpc_client
from wordpress_xmlrpc.methods import media, posts

#import importlib
#importlib.reload(sys)
#sys.setdefaultencoding('utf-8') #python3下不支持该用法

class Handler(BaseHandler):
    crawl_config = {
    }
    
    def __init__(self):#继承Deal类
        self.deal = Deal()
        
    @every(minutes=24 * 60)
    def on_start(self):
        self.crawl('https://www.xxxxxx.com/news/index.html', callback=self.index_page)

    @config(age=10 * 24 * 60 * 60)
    def index_page(self, response):
        for each in response.doc('h3 > a').items():
            self.crawl(each.attr.href, callback=self.detail_page)

    @config(priority=2)
    def detail_page(self, response):
        title = response.doc('h1').text(),
        tmp_text = response.doc('.loadimg.fadeInUp > p').text(),
        tmp_text = ''.join(str(tmp_text)) #将元组数据转换为字符串
        tmp_text = tmp_text.replace("下面我们一起来看看她最新的番号作品吧！","") 
        new_text = tmp_text.replace("('  ","")#清洗掉原文中的标记
        
        for each in response.doc('.loadimg.fadeInUp > p > img').items():
           img_url = each.attr.src        
           if ("xxxxxx" in img_url): #过滤掉不在xxxxxx站点上的图片连接
               split_url = img_url.split('/')
               dir_name = split_url[-2] + '/'
               dir_path = self.deal.mkDir(dir_name)
               file_name = split_url[-1]
               relativpath = '<img src="/downimages/' + dir_name + file_name + '"><br>'#构建图片显示的相对路径
               new_text = relativpath + new_text #将图片插入文章头部
               self.crawl(img_url,callback=self.save_img, save={'dir_path':dir_path ,'file_name':file_name})
        title = ''.join(str(title)) 
        title = title.replace("('","")
        title = title.replace("',)","")
        wp = Client('http://server/xmlrpc.php', 'username', 'password')
        post = WordPressPost()
        post.title = title
        post.content = new_text
        post.post_status = 'publish'
        post.id = wp.call(posts.NewPost(post))
        #print("爬取注入:"+ post.id + "post.title")
     
   

    def save_img(self,response): #保存图片
        content = response.content
        dir_path = response.save['dir_path']
        file_name = response.save['file_name']
        file_path = dir_path + '/' + file_name 
        self.deal.saveImg(content,file_path)
 
import os

class Deal:
    def __init__(self):
        self.path = DIR_PATH
        if not self.path.endswith('/'):
            self.path = self.path + '/'
        if not os.path.exists(self.path):
            os.makedirs(self.path)

    def mkDir(self, path):
        path = path.strip()
        dir_path = self.path + path
        exists = os.path.exists(dir_path)
        if not exists:
            os.makedirs(dir_path)
            return dir_path
        else:
            return dir_path

    def saveImg(self, content, path):
        f = open(path, 'wb')
        f.write(content)
        f.close()

    def saveBrief(self, content, dir_path, name):
        file_name = dir_path + "/" + name + ".txt"
        f = open(file_name, "w+")
        f.write(content.encode('utf-8'))

    def getExtension(self, url):
        extension = url.split('.')[-1]
        return extension

#        return {
#            "title": title,
#            "contenttext": contenttext,
#            
#        }

```