---
layout: post
title:  "通过upyun解决iOS7.1后使用AdHoc下载的问题"
date:   2015-09-01 13:13:13
categories: jekyll update
---
## 前言

相信大家都已经知道，在苹果发布iOS7.1后，AdHoc不能够接受未经ssl验证的mainfest了，也就是说plist文件的路径需要从http转换为https了。

	itms-services://?action=download-manifest&url=http://yourdomain.com/app.plist
	==> //更换为
	itms-services://?action=download-manifest&url=https://yourdomain.com/app.plist

应对这个改变，我们有两种方法可以采取：

- 在本地局域网搭建ssl环境。之前的博客[Mac OS X 中为Apache开启ssl]()就是为了应对这个改变做得一些尝试，但是无奈在本地局域网中配置证书太过麻烦，最终放弃。想要尝试该方法的童鞋可以参考[这篇博客](http://blog.cnbluebox.com/blog/2014/03/25/ios7-dot-1xia-shi-yong-adhocfang-fa-xia-zai-de-jie-jue-fang-an/)
- 采用https外链。很高兴了解到[upyun]()今天（2015.9.1）宣布全面免费，并且upyun是支持做https外链的。所以今天主要尝试通过upyun做一个支持ssl的外链。

仔细想想，是不是今后博客中的图片都可以放在upyun做CDN加速了，吼吼。

## 准备工作

注册upyun成功后会提示必须绑定支付宝做实名认证，否则无法享受全部功能，吓得我赶紧连接支付宝做了实名认证，没有做实验不实名认证的话，是否支持https外链功能。

添加一个`存储类`空间，并为这个空间开启https访问功能，如下图：

![](http://redharry.b0.upaiyun.com/pic/upyun_with_ssl.gif)

哈哈，其实这么简单地步骤完全没有必要上图，我只是单纯想测试下Mac上录制gif图片（我用的是[Recordit](http://www.recordit.co/)这个软件）以及测试下upyun的CDN图片加速。

## 上传文件

上传文件需要先创建一个`操作员`。在`账号管理`菜单下选择`操作员管理`来新建一个操作员，并在`通用`菜单中选择`操作员管理授权`将刚才新建的操作员授权于管理空间权限。

假设现在我们有一个名为`bucket`的空间，并且为这个空间授权了一个名为`user`的操作员。那么：

- 关于FTP的配置

	- 主机：v1.ftp.upyun.com (电信) v2.ftp.upyun.com (联通网通) v3.ftp.upyun.com (移动铁通) v0.ftp.upyun.com (自动判断
	- 用户：操作员的用户名/空间名（对应我们的空间应该是`user/bucket`）
	- 密码：操作员的密码
	- 端口：21
	- 文件传输协议：FTP

- 关于外链的配置

	- 默认域名：bucket.b0.upaiyun.com （bucket是空间名）
	- 文件访问方式：http://默认域名/文件路径
（例如：文件路径/dir/pic.jpg 该图片对外访问地址为：http://bucket.b0.upaiyun.com/dir/pic.jpg）
	- ssl方式只需将http替换为https
	- 如果绑定了自己的域名，只需将默认域名替换为自己的域名就可以了



如果是使用客户端的话，推荐使用[FileZilla]()这个软件，不要再提Finder的远程连接服务器了，功能弱爆了。

成功登陆后的软件界面大概是这个样子：

![](http://redharry.b0.upaiyun.com/pic/how_does_filezilla_looks_like.png)

可以直接把文件拖到红框圈住的地方实现文件的上传。

如果非要使用终端的话，下面是终端的操作：

	// 以`user/bucket`这个用户登录ftp服务器
	ftp user/bucket@v1.ftp.upyun.com
	// 然后输入密码
	// 登录成功
	// 查看所有可用命令
	？
	// 上传本地文件
	put Info.plist /plist/abc.plist //将本地的工作目录下地Info.plist 上传至服务器/plist/abc.plist这个路径，plist这个文件夹如不存在，会自动创建
	// 下载远程文件
	get /plist/abc.plist abc.plist //将远程服务器/plist/abc.plist 拷贝到本地工作目录下
	
至此，就可以通过`https://bucket.b0.upaiyun.com/plist/abc.plist`这个外链来访问plist文件了。







	

