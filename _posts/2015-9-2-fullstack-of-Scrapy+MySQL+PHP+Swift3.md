---
layout: post
title:  "Scrapy+MySQL+PHP+Swift开发攻略系列（三）数据库MySQL篇"
date:   2015-08-29 09:09:09
categories: jekyll update
tags:
  - Linux
  - Scrapy
  - Swift
---

##系列目录

你可以从这个地方做一个快速跳转。

- [Scrapy+MySQL+PHP+Swift开发攻略系列（一）之前言篇](http://blog.coderharry.com/2015/08/08/fullstack-of-Scrapy+MySQL+PHP+Swift.html)
- [Scrapy+MySQL+PHP+Swift开发攻略系列（二）之爬虫篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（三）之数据库MySQL篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（四）之爬虫被封+爬虫自动运行篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（五）之API篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（六）之RESTAPI篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（七）之Swift篇]()


## 定义管道文件

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

然后到`setting.py`这个文件里面加上这么一行
	
	ITEM_PIPELINES = ['dbmeizi.pipelines.PrintPipeline',] 

是指定爬虫爬取后通过`PrintPipeline`这个管道输出内容。所以这两个名字要写一致。

## 运行爬虫
到现在为止，这个爬虫就可以正常工作了。在工程的根目录下执行如下命令：

	scrapy crawl dbmeiziSpider



大功告成。

## 最后

按照惯例，放上源码地址：




