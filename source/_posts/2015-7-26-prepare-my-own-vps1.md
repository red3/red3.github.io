---
layout: post
title:  "折腾自己的服务器系列（一）之配置服务器"
date:   2015-07-26 11:13:36
tags:
  - Linux
---

生命不息，折腾不止。为了消灭自己的懒惰情绪，增强匮乏感，也正值阿里云服务器打折，遂买了个服务器折腾一下。在这个过程中所遇到的所有问题、难点等均可在网路上找到相关资料，之所以还再写这个系列，一是记录下这个过程，方便以后查询；二是因为网路中很多资料都已过时，故整理出最新可用的资料供需要的人查询。

So，这个系列大概要做这些事情：

- 配置服务器：包括Apache，PHP，MYSQL，FTP，PhpMyAdmin等
- 配置域名：将自己的域名指向服务器，以及设置二级域名等。

今天这个系列主要就是记录写如何配置刚购买的服务器。
这里需要说明的是，以下的操作都是基于CentOS6.4版本。

## 安装基本的软件服务

首先通过远程登录，连接到服务器，因为我用的是Mac，所以直接用ssh命令登录：

	ssh ip地址 -l root 
	

CentOS下推荐适用yum安装软件，故通过下列命令安装我们所需的服务：
	
	// 安装Apache服务
	yum install -y httpd
	// 安装PHP
	yum install -y php
	// 安装MYSQL
	yum install -y mysql-server mysql
	// 安装sftp
	yum install -y vsftpd 
	
至此，基本的服务都已安装，不得不感叹Linux作为服务器来说，确实比MAC和Windows有优势，安装太方便了。接下来的就是配置下这些服务，让他们运行起来。

## 配置Apache

我们需要修改两个地方，修改配置文件以及在网站根目录创建index.html.
	
	// 下面这个目录是配置文件所在目录
	vi /etc/httpd/conf/httpd.conf
找到 `#ServerName www.example.com:80`将注释关闭，变成类似这样的`ServerName 服务器的公网ip:80`。

	// 下面这个目录是Apache默认的网站目录
	cd /var/www/html
	touch index.html
	
创建`index.html`后，往里面随便写点东西。
现在在本地浏览器中访问服务器的公网ip，应该就可以看到刚才我们在`index.html`中输入的内容了。
需要注意的是阿里云服务器默认是开放**80**端口的，通过这个命令`service iptables status`查看当前防火墙状态:

	service iptables status
	表格：filter
	Chain INPUT (policy ACCEPT)
	num  target    prot opt source            destination         
	1    ACCEPT    tcp  --  0.0.0.0/0         0.0.0.0/0    tcp dpt:22 
	2    ACCEPT    tcp  --  0.0.0.0/0         0.0.0.0/0    tcp dpt:80
	
可以看到当前服务器已经开放了**80**端口和**22**端口。
如果服务器默认没有开放**80**端口，还需要开放**80**端口，可以这样：
	
	// 开放80端口 
	iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
	// 因为要使用sftp，顺便也开放22端口
	iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
	// 保存
	service iptables save

至此，Apache的设置就完成了。

## 配置SFTP

默认的，和Apache一样，sftp的配置文件在这个目录`/etc//vsftpd/`下。sftp默认是不允许root用户登录的，因为是我自己的服务器，并不需要别人参与开发，所以就简单的设置下如何配置root用户可以登录（需要说明的是开启root用户会很不安全，因为root用户的权限太大）：

	// ftpusers中的用户是黑名单用户是不允许从sftp登录的 
	vi /etc/vsftpd/ftpusers // 把列表中的root注释
	// sftp默认开启userlist_deny，会阻止user_list中的用户
	vi /etc/vsftpd/user_list //  把列表中的root注释
	 
	
现在尝试在客户端通过sftp连接服务器吧。对于Mac电脑，推荐这个软件[Filezilla](https://filezilla-project.org/download.php?type=client)，或者给Sublime装个插件，都可以连上服务器了。

## 配置MySQL

之前的这个步骤`yum install -y mysql-server mysql`已经安装了MySQL所需的软件服务。要运行MySQL服务，需要这些命令：
	
	// 启动mysqld服务 要关闭的话将start替换成stop
	service mysqld start
	// 给root用户设置密码为ab123 默认刚安装的数据库是没有密码的，所以需要先设置密码
	mysqladmin -uroot -password ab123
	// 连接数据库
	mysql -uroot -p
	// 提示输入密码后，就可进入mysql命令模式了
	// 推出mysql命令模式
	// exit
	
关于数据库的管理，还是推荐使用phpMyAdmin更加方便些。关于phpmyadmin，可以选择yum安装，也可以直接从[官网](http://www.phpmyadmin.net/downloads/)下载。
今天的系列就先到这了，下一系列说说配置域名的事情。






