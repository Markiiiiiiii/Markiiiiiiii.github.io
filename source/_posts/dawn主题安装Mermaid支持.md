---
title: Hexo dawn主题安装Mermaid支持
date: 2020-08-24 18:22:35
tags:
- hexo
categories:
- blog
---
## Mermaid介绍
mermaid是一个非常优秀的JS渲染流程图、甘特图、时序图的JS库可以非常方便的嵌入到各种的Markdow编辑器之中。

网上也有很多教程教大家将mermaid嵌入到hexo中，但是大部分的教程都是基于hexo的主题是.swig作为模板文件的。

本blog使用的是[dawn](http://cofess.github.io/)主题，这个主题非常的简介而且集成了很多的功能，可以看出作者还是非常的用心的，但是作者使用的.ejs作为模板。这样就使得嵌入mermaid的时候有了些许的不同。

### 第一步 npm 安装 mermaid
```
npm install hexo-filter-mermaid-diagrams --save
或
cnpm install hexo-filter-mermaid-diagrams --save
```

### 第二步 手动增加一个 mermaid.ejs文件
```
在themes/dawn/layout/_script/下新建一个mermaid.ejs文件
内容为
<% if (theme.mermaid.enable) { %>
    <script src="https://unpkg.com/mermaid@<%- theme.mermaid.version %>/dist/mermaid.min.js"></script>
  <% } %>
```
### 第三步 修改主题目录下的_config.yml文件
```
文件末尾增加

  # mermaid chart
mermaid: ## mermaid url https://github.com/knsv/mermaid
  enable: true  # default true
  version: "8.7.0" # default v7.1.2,可以手动修改为最新的版本，不然还是按照默认7.1.2版本。
  options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js
       #startOnload: true  // default true
```

### 第四步 重新生成页面
```
hexo clean
hexo g
hexo d 
```
### Tips.注意mermaid的语法问题,一定要使用   

    ``` mermaid 包裹