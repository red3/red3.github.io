---
layout: post
title:  "Scrapy+MySQL+PHP+Swift开发攻略系列（三）数据库MySQL篇"
date:   2015-09-03 09:09:09
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

安装过程中这些情况你可能需要参考：

- 修改`site.cfg`这个文件：

		// 将下面这行代码的注释打开，并将后面的路径修改成本地mysql路径
		mysql_config.path = "/usr/local/mysql/bin/mysql_config"
		
- 安装完成后`import MySQLdb`报错提示：`Library not loaded: libmysqlclient.18.dylib	`：

		// 这种情况需要把mysql的该动态库链接到用户库目录中去
		sudo ln -s /usr/local/mysql/lib/libmysqlclient.18.dylib /usr/lib/libmysqlclient.18.dylib

## 配置MySQL

在`settings.py`配置文件中加入数据库的配置：

	# start MySQL database configure setting
	MYSQL_HOST = 'localhost'
	MYSQL_DBNAME = 'dbmeizi'
	MYSQL_USER = 'root'
	MYSQL_PASSWD = 'hali'
	MYSQL_PORT = 3306
	# end of MySQL database configure setting

需要注意检查端口要跟实际的数据库服务的端口相一致。

通过`phpMyAdmin`连接数据库创建一个`dbmeizi`数据库，一个`meizi`表，用来保存爬虫数据：
	
	// 创建数据库
	create database dbmeizi;
	// 创建表
	drop table if exists `meizi`;
	create table `meizi` (
	`id` int(11) unsigned not null auto_increment comment 'id',
	`title` varchar(100) not null comment '标题',
	`imgsrc` varchar(200) not null comment '图片链接',
	`topic_link` varchar(100) not null comment '主题链接',
	`star_count` int(11)  not null  comment '点赞数',
	`update_time` int(11) not null comment '状态更新时间',
	primary key (`id`)
	) engine = innodb default charset = utf8 comment = '豆瓣妹子表';

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

``` python

import scrapy
import MySQLdb
import MySQLdb.cursors
from twisted.enterprise import adbapi

class DbmeiziPipeline(object):
    def __init__(self, dbpool):
	   self.dbpool = dbpool

    @classmethod
    def from_settings(cls, settings):
	   dbargs = dict(
		  host = settings['MYSQL_HOST'],
		  db = settings['MYSQL_DBNAME'],
		  port = settings['MYSQL_PORT'],
		  user = settings['MYSQL_USER'],
		  passwd = settings['MYSQL_PASSWD'],
		  charset = 'utf8',
		  cursorclass = MySQLdb.cursors.DictCursor,
		  use_unicode = True,
		)
	   dbpool = adbapi.ConnectionPool('MySQLdb', **dbargs)
	   return cls(dbpool)

    #pipeline默认调用
    def process_item(self, item, spider):
        d = self.dbpool.runInteraction(self._do_upinsert, item, spider)  
    	return item
    #将每行更新或写入数据库中
    def _do_upinsert(self, conn, item, spider):                 

    	valid = True
    	for data in item:
    		if not data:
    			valid = False
    			# raise DropItem("Missing {0}!".format(data))
    			# print "Missing data"
        if valid:
            result = conn.execute("""
                insert into meizi(title, imgsrc, topic_link, star_count, update_time) 
                values(%s, %s, %s, %s, %s)
                """, (item['title'], item['imgsrc'], item['topic_link'], item['star_count'], item['update_time']))
            if result:
                print "added a girl into db"
            else:
                print "failed insert into meizi"

```

`from_settings`这个方法中我们创建了数据库连接，`process_item `方法里面将每一个item存入数据库。主要是常用数据库的操作，不细说了。


## 启用一个Item Pipeline组件

为了启用一个`Item Pipeline`组件，必须将它的类添加到`ITEM_PIPELINES`配置，具体就是到`setting.py`这个文件里面加上这么一行：
	
	ITEM_PIPELINES = ['dbmeizi.pipelines. DbmeiziPipeline',] 

是指定爬虫爬取后通过哪个管道输出内容。

## 运行爬虫
到现在为止，这个爬虫就可以将数据保存到数据库了。在工程的根目录下执行如下命令：

	scrapy crawl dbmeiziSpider



大功告成。

## 最后

按照惯例，放上源码地址：




