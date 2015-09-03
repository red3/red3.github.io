---
layout: post
title:  "Scrapy+MySQL+PHP+Swift开发攻略系列（三）数据库MySQL篇"
date:   2015-09-03 09:09:09
categories: jekyll update
tags:
  - Linux
  - Scrapy
  - Swift
---

##系列目录

你可以从这个地方做一个快速跳转。

- [Scrapy+MySQL+PHP+Swift开发攻略系列（一）之前言篇](http://blog.coderharry.com/2015/08/08/fullstack-of-Scrapy+MySQL+PHP+Swift1.html)
- [Scrapy+MySQL+PHP+Swift开发攻略系列（二）之爬虫篇](http://blog.coderharry.com/2015/08/08/fullstack-of-Scrapy+MySQL+PHP+Swift2.html)
- [Scrapy+MySQL+PHP+Swift开发攻略系列（三）之数据库MySQL篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（四）之爬虫被封+爬虫自动运行篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（五）之API篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（六）之RESTAPI篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（七）之Swift篇]()

## 温故而知新

上个系列中，`spider`已经爬取到我们感兴趣的`item`了。这个系列我们主要实现怎么把抓取到的`item`保存到`MySQL`数据库中。

## 安装MySQLdb

安装`MySQLdb`从这个[网站](https://pypi.python.org/pypi/MySQL-python/1.2.5#downloads)采用源码安装。

## Why use Item Pipeline

当得到`item`的时候，我们需要验证`item`中的某些字段是否合法，是否存在重复的`item`，然后才将合格的`item`保存到数据库。

这就是`item pipeline`的典型使用场景。

当`item`在`Spider`中被收集之后，将会被传递到`item pipeline`，每个`item pipeline`组件是实现了简单方法的Python类。他们接收到Item并通过它执行上面提到的典型行为。

## 编写我们的 Item Pipeline

每个`item pipeline`组件是一个独立的Python类，必须实现以下方法:

	process_item(self, item, spider)
	每个item pipeline组件都需要调用该方法，这个方法必须返回一个 Item (或任何继承类)对象， 或是抛出 DropItem 异常，被丢弃的item将不会被之后的pipeline组件所处理。

	参数:	
	item (Item 对象) – 被爬取的item
	spider (Spider 对象) – 爬取该item的spider


打开`pipelines.py`，这个文件应该是这个样子：

{% highlight python %}

from scrapy.conf import settings
from scrapy.exceptions import DropItem
from scrapy import log

class PrintPipeline(object):
    """just prit the item"""
    def __init__(self):
        
    def process_item(self, item, spider):
        print """
                meizi(title, imgsrc, topic_link, star_count, update_time) 
                values(%s, %s, %s, %s, %s)
            """, (item['title'], item['imgsrc'], item['topic_link'], item['star_count'], item['update_time'])
        return item

{% endhighlight %}

`process_item`这个方法中我们暂时只是将item打印输出，看是不是符合我们的数据要求。



## 启用一个Item Pipeline组件

为了启用一个`Item Pipeline`组件，必须将它的类添加到`ITEM_PIPELINES`配置，具体就是到`setting.py`这个文件里面加上这么一行：
	
	ITEM_PIPELINES = ['dbmeizi.pipelines.PrintPipeline',] 

是指定爬虫爬取后通过哪个管道输出内容。
## 运行爬虫
到现在为止，这个爬虫就可以正常工作了。在工程的根目录下执行如下命令：

	scrapy crawl dbmeiziSpider



大功告成。

## 最后

按照惯例，放上源码地址：




