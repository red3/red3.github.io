---
layout: post
title:  "Scrapy+MySQL+PHP+Swift开发攻略系列（四）之爬虫被封+爬虫自动运行篇"
date:   2015-09-08 11:11:11
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

## 防止爬虫被封

系列（二）中提到过，对于一个网络请求，网站后台可能会检查其`User-Agent`，所以一个防止爬虫可行的做法就是设置请求的`User-Agent`。对于频繁的请求，还要对`User-Agent`做随机变换。

可以通过`Downloader Middleware`来修改爬虫的request和respons。

### 使用下载器中间件(Downloader Middleware)

要激活下载器中间件组件，需要将其加入到`DOWNLOADER_MIDDLEWARES`设置中。 该设置是一个字典(dict)，键为中间件类的路径，值为其中间件的顺序(order)。

	DOWNLOADER_MIDDLEWARES = {
    'dbmeizi.middlewares.RandomUserAgent': 1
	}

因为要使用随机的`User-Agent`，所以在设置中，顺便加上随机`User-Agent`的配置：
	
	USER_AGENTS = [
    "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5",
    "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.8) Gecko Fedora/1.9.0.8-1.fc10 Kazehakase/0.5.6",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/535.20 (KHTML, like Gecko) Chrome/19.0.1036.7 Safari/535.20",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.85 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/600.7.12 (KHTML, like Gecko) Version/8.0.7 Safari/600.7.12",
	]
	
### 编写下载器中间件

创建一个中间件文件`middlewares.py`.

``` python

import random
import base64

class RandomUserAgent(object):

    def __init__(self, agents):
        self.agents = agents

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.settings.getlist('USER_AGENTS'))

    def process_request(self, request, spider):
        request.headers.setdefault('User-Agent', random.choice(self.agents))
        
```


## 自动运行

为了源源不断获取数据，要每天都运行爬虫抓取数据，显然每次都人为发起爬虫是不切实际的。

在Mac上可以通过`crontab`让爬虫定时自动执行。

	// 为当前用户新增任务
	crontab -e
	// 增加如下记录 注意替换自己的爬虫目录 由于环境变量的原因，scrapy要给出全路径
	0 10 * * * cd /Users/herui/Sites/dbmeizi && /usr/local/bin/scrapy crawl dbmeiziSpider
	
如上，添加了一个任务，这个任务会每天早上10：00启动，这个任务要做得就是进入爬虫目录，并启动爬虫。

不过查了下，Mac上目前好像不太推荐用`crontab`了，推荐使用`launchd`. 有机会可以了解下。


## 最后

按照惯例，放上源码地址：




