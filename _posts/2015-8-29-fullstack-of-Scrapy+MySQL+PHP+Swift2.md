---
layout: post
title:  "Scrapy+MySQL+PHP+Swift开发攻略系列（二）爬虫篇"
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

##安装Scrapy
爬虫框架我选择用Python写的Scrapy。

当然准备工作是确保你的Mac安装了`commandline`和`pip`.

- 安装`commandline`可以通过直接安装`Xcode`或者在终端运行`xcode-select --install`命令安装。
- 安装`pip`：遵从官方的这个[步骤](https://pip.pypa.io/en/stable/installing.html#install-pip)

然后通过`pip`安装`Scrapy`.
	
	sudo pip install scrapy
	
如果你安装成功，请直接跳到下一节。我在公司电脑的环境（OS X 10.10 Python2.7.10）以及家里的电脑的环境（OS X 10.11 Python2.7.10）下安装会因为类似`libxml not found`的原因失败，通过以下方式解决：

	brew install libxml2
	brew install libxslt
	brew link libxml2 --force
	brew link libxslt --force

安装成功以后，如果`scrapy startproject xxx`报类似的错`ImportError: cannot import name xmlrpc_client`，通过以下方式解决：

	sudo rm -rf /Library/Python/2.7/site-packages/six*
	sudo rm -rf /System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/six*
	sudo pip install six

如果以上方法不能失效，请参考[这个链接](http://stackoverflow.com/questions/30964836/scrapy-throws-importerror-cannot-import-name-xmlrpc-client)
##开始爬虫

通过下面这个命令生成一个爬虫工程：

	scrapy startproject spiders
	
然后我们会看到`scrapy`已经为我们生成了一个工程。这个工程大概是这个结构
	
	dbmeizi
	|__ __init__.py
	|__ items.py
	|__ pipelines.py
	|__ settings.py
	|__ spiders

下面分别解释下各个文件：

`items.py` - item相当于是mvc中的model，在items里我们定义了自己需要的模型
`piplines.py` - pipline俗称管道，这个文件主要用来把我们获取的item类型存入MySQL
`settings.py` -  在这个文件里面配置整个工程的一些设置。例如MySQL的数据库名，数据库地址和数据库端口号等等。
`spiders` - 这个文件夹存放爬虫文件。

至此，我们就可以正式开始我们的编码工作了。

## 定义Model层

首先我们想确定一个网站上的图片包含哪些信息，要解决这个问题，就需要打开这个网页使用`开发者工具`(快捷键`option+command+i`), 使用`页面选择器`(开发者工作左上角的放大镜图标)选择一张图片，效果如下:

![](/assets/2015/img_spiders01.png)


可以看出，我们选中的`div`块中包含了我们想要的最基本的资料。这个过程，其实就是我们爬虫的一个工作原理，通过网页元素找到我们想要的内容，只不过现在我们是手动查找，等发现规律，我们就通过爬虫程序自动爬取内容。

所以item.py里面是这个样子：

	from scrapy.item import Item, Field

	class MeiziItem(Item):
	
		imgsrc = Field()
    	title = Field()
    	topic_link = Field()
    	star_count = Field()
    	update_time = Field()
    	
相当于我们继承自类Item创建了我们自己的MeiziItem，然后我们的自定义类有5个属性，`star_count`是设计用来让用户点赞的，最后的`update_time `可以用来记录修改时间。

## 编写爬虫

在`spiders`文件夹下新建`dbmeizi_scrapy.py`文件。
这个文件里面是这个样子：

{% highlight python %}

from scrapy import Spider
from scrapy.selector import Selector
from dbmeizi.items import MeiziItem
import time

class dbmeiziSpider(Spider):
    name = "dbmeiziSpider"
    allowed_domin =["dbmeinv.com"]
    strArray = []
    for i in range(1, 3, 1):
        str = "http://www.dbmeinv.com/?pager_offset=%d" % i
        strArray.append(str)
    start_urls = strArray
            
    def parse(self, response):
        divResults = Selector(response).xpath('//div[@class="img_single"]')
        for div in divResults:
            item = MeiziItem()
            href = div.xpath('.//a')[0]
            img = div.xpath('.//img')[0]
            item['topic_link'] = href.xpath('@href').extract()[0]
            item['title'] = img.xpath('@title').extract()[0] 
            item['imgsrc'] = img.xpath('@src').extract()[0]
            item['star_count'] = 0
            item['update_time'] = time.time()
            yield item
    
{% endhighlight %}

需要解释的几点概念：

- `allowed_domin ` - 指定在哪个网站爬东西
- `start_urls` - 需要爬取哪些页面
- `parse `方法 - 继承自父类，可以想象成这个方法一开始拿到的数据就是整个网页的html代码，我们要通过各种过滤，拿到最终我们感兴趣的内容。

最终，爬虫通过上面的代码爬到我们感兴趣的内容了，通过这些内容为`item`赋值，`item`会通过管道文件输出，所以接下来我们要创建管道文件。

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




