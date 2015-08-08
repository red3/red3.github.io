---
layout: post
title:  "Scrapy+MySQL+PHP+Swift开发攻略系列（一）之前言篇"
date:   2015-08-08 22:22:33
categories: jekyll update
tags:
  - Linux
  - Scrapy
  - Swift
---

##系列目录

你可以从这个地方做一个快速跳转。

- [Scrapy+MySQL+PHP+Swift开发攻略系列（一）之前言篇]({{ page.url }})
- [Scrapy+MySQL+PHP+Swift开发攻略系列（二）之爬虫篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（三）之数据库MySQL篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（四）之爬虫被封+爬虫自动运行篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（五）之API篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（六）之RESTAPI篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（七）之Swift篇]()

##缘由

前端时间读到[叶孤城___ ](http://www.jianshu.com/users/b82d2721ba07/latest_articles)的博客，有个系列文档是讲怎么实现从服务端后台到移动端App的整个过程，看了很受触动。真的非常感谢iOS业界有这些经常将自己的技术实践分享出来的前辈，从他们的博客中可以收获很多东西。之所以还要写这个系列，是因为在[叶孤城___ ](http://www.jianshu.com/users/b82d2721ba07/latest_articles)的博客文章中，是通过Mongodb数据库将数据存储起来的，在接口层是通过Python实现的。佳缘的后台是通过MySQL+PHP实现的，所以便打算将爬虫里面的Mongodb存储替换成MySQL，在接口层替换Python通过PHP操作MySQL。之前的博客中有提到过，我最近有买了一台服务器，所以顺便学习了将爬虫部署在服务器上，以及爬虫的自动运行等。

这个系列的文档仅是做为入门的一个实现，毕竟之前也没有做过Python的开发。随着技术的深入了解，我会随时补充一些东西。

##这个系列到底做些什么

如文章标题所说，这个系列我主要通过爬虫爬了一家网站的妹子的照片，然后把照片相关信息存储到了MySQL数据库。然后把这个爬虫部署在服务器上（没有服务器的话在自己的电脑上也是可以的，保证电脑长时间是开机状态即可）让爬虫自动运行。这样我们就有了持续不断的内容了。接着在接口层，要开发一个接口，提供给客户端每天的最新照片信息，以及给照片点赞的接口。在客户端层面，要展示一组图片列表，要实现客户端的点赞功能跟服务器的交互。客户端会通过图片的点赞数量，选出当日热门图片。客户端主要会采用Swift编写。

目前为止，这个项目已经进展到了接口阶段，服务器的数据库上每天都会有新的信息保存进来。接下来的时间，我会把已经完成的部分抽空更新出来。


