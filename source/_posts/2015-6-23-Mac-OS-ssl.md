---
layout: post
title:  "Mac OS X 中为Apache开启ssl"
date:   2015-06-23 22:19:36
categories: jekyll update
---

最近由于工作需要，需要给Mac本地的Apache配置ssl环境，折腾了一下午，终于解决了，把过程记录下来，留给有需要的人。

- 生成ssl证书

1. 生成主机密匙 （'ssl'这个文件夹可以随意起名字，只要在后面的设置中保持一致即可）


		sudo mkdir /private/etc/apache2/ssl
		cd /private/etc/apache2/ssl
		sudo ssh-keygen -f server.key


2. 生成证书请求文件（这个过程感觉跟iOS生成开发证书类似）

		sudo openssl req -new -key server.key -out request.csr
    	// 这个过程中会让输入一些证书机构的信息，按照提示或者留空就行
   
    
3. 使用server.key 和 request.csr 生成ssl证书

		sudo openssl x509 -req -days 365 -in request.csr -signkey server.key -out server.crt

- 配置Apache

1. 编辑httpd.conf文件

    sudo vi /private/etc/apache2/httpd.conf

    
去掉下面四行前面的 '#'

``` php
LoadModule ssl_module libexec/apache2/mod_ssl.so
LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
Include /private/etc/apache2/extra/httpd-ssl.conf
Include /private/etc/apache2/extra/httpd-vhosts.conf
``` 

2. 编辑httpd-ssl.conf文件

    sudo vi /private/etc/apache2/extra/httpd-ssl.conf

将以下两行：

    SSLCertificateFile "/private/etc/apache2/server.crt"

    SSLCertificateKeyFile "/private/etc/apache2/server.key"

    
分别修改为：（需要注意的是ssl文件夹为第1步创建的文件夹）

    SSLCertificateFile "/private/etc/apache2/ssl/server.crt"

    SSLCertificateKeyFile "/private/etc/apache2/ssl/server.key"

需要注意的是该文件68行的代码会在Apache运行的时候报错

    SSLMutex  "file:/private/var/run/ssl_mutex"

查找网上的解决方案是将这行代码注释，不知道会不会带来安全隐患，烦请知道的同学告知。

3. 编辑httpd-vhosts.conf文件

在文件末尾添加：

    <VirtualHost *:443> 
    SSLEngine on
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /private/etc/apache2/ssl/server.crt 
    SSLCertificateKeyFile /private/etc/apache2/ssl/server.key
    ServerName localhost
    // copy your DocumentRoot setting in httpd.conf to here
    DocumentRoot "/Users/$yourName/Sites"
    </VirtualHost>

这时需要注意，httpd-vhosts.conf文件里面有个默认的 <VirtualHost *:80>标签， 不做修改的话，访问localhost会报错:The Request / Not Found

这是因为<VirtualHost *:80>里的设置会覆盖以前单站点模式下的设置, 所以要把该文件里的一些设置同步过来, 我把他修改成这样:


	<VirtualHost *:80>
	ServerAdmin xxx@example.com
	// copy your DocumentRoot setting in httpd.conf to here
	DocumentRoot "/Users/$yourName/Sites"
	ServerName localhost
	ErrorLog "/private/var/log/apache2/error_log"
	CustomLog "/private/var/log/apache2/access_log" common
	</VirtualHost> 

进行到这里就配置完了，重启Apache，访问https://localhost试试吧！

    sudo apachectl configtest
    sudo apachectl restart

- 后记

其实最近折腾这个主要是为了练习iOS应用框架中使用SSL加密以及实现iOS自动打包至服务器使用itms-service协议自动安装至手机的过程，然而由于服务器缺少https环境，本地https又缺少SA认证，尚未实现，权当给自己挖个坑，钻研钻研吧！

参考：

[http://www.cnblogs.com/y500/p/3596473.html](http://www.cnblogs.com/y500/p/3596473.html)

[http://stackoverflow.com/questions/13969272/apache-sslmutex-issue](http://stackoverflow.com/questions/13969272/apache-sslmutex-issue)



