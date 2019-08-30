---
layout: post
title:  "nodejs和js的前端后端开发笔记"
date:   2019-08-28 15:24:54
categories: web
tags: nodejs 原创
author: wuxy
---

* content
{:toc}

## 常见问题

### nodejs是什么？
- js和nodejs本质上是不同的东西，js是一种编程语言，而nodejs是一种运行环境，nodojs的出现使得js可以开发服务器应用。但是在nodejs中会有一些超出js语法的使用，比如require。而js本身不支持import,include等操作，js想要实现调用其他js文件，可以使用<script>标签。
- https://blog.csdn.net/Inuyasha1121/article/details/51071803 ：
js文件中引用其他js文件，但文中原理基本也是使用<script>

### 服务器应用开发
- 服务器应用开发也就是后端开发，理论上，任何可以在PC上开发应用的语言的是可以的，比如java,C#,python，等等，那么C，C++可以吗？当然可以的，只不过那些专用的服务器语言具有很好的开发生态和相关库，便于和前端交互，所以主流的后端开发语言一般都是php,java,C#等，而有了nodejs以后，js也可以开发后端应用了。

### 前端如何读取mysql数据并显示在页面上？
- html不能读取本地数据，也不能跨域读取；
- 但是html可以嵌入js语言，jQuery可以用ajax读取服务器端（或本地）的json数据；
- https://bbs.csdn.net/topics/390197089 ：javascript读取mysql数据库的数据；
- 最好的方式是，用服务器语言来操作数据库，然后前后端通过请求来实现数据传递，传递的数据格式可以是json.


## 扩展阅读
 - https://www.cnblogs.com/willian/p/4195583.html ：NodeJS让前端与后端更友好的分手，提出了一些前后端的开发方案
 - https://blog.csdn.net/mqy1023/article/details/51194823 ： web实战(三)— — Tab选项卡切换效果
