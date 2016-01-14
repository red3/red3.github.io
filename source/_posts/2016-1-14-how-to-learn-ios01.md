
---
layout: post
title:  "iOS学习沙龙系列（一）：开篇"
date:   2016-1-14 09:00:00
---


## 前言


- 为什么学习iOS：学习是投资回报率最高的行为，因为所习得的技能，认识了另一个世界
	- PHP：
	- Android：
	- QA：
	- Others（人人都应该学习一门编程语言，畅销书作家李笑来，初中的时候处于兴趣爱好学习了Basic程序语言。后来当他在新东方教书的时候，利用自己的计算机知识，编写了批处理脚本，在短时间完成了海量的工作，写成了《TOEFL 核心词汇21天突破》）

 
- iOS也应该学习其他语言


- 课程简介：10课时，每课时30mins，总计约5h。对于iOS来说，入门需要1个月，上手需要3个月。

## 纲要
### 学习材料的选择

1. 硬件条件：Macbook pro或者黑苹果，黑苹果建议淘宝找专业人士花钱解决
2. 软件条件：Xcode
3. 教程视频的选择：不太建议跟着国内培训机构的视频走：低效、过时（目前最新是Xcode7，iOS9）、甚至有错误的东西
4. 经典书籍：后文会专门讲

### iOS、Mac OS X系统简介

1. iOS、OS X文件系统

* OS X文件系统

```
drwxrwxr-x+ 73 root  admin  2482  1 12 09:15 Applications
drwxr-xr-x+ 62 root  wheel  2108 12  7 16:27 Library
drwxr-xr-x@  2 root  wheel    68 10 15 09:17 Network
drwxr-xr-x@  4 root  wheel   136 12  7 16:28 System
drwxr-xr-x   5 root  admin   170 10 15 09:17 Users
drwxrwxrwt@  3 root  admin   102  1 12 09:28 Volumes
drwxr-xr-x@ 39 root  wheel  1326 12  7 16:28 bin
drwxrwxr-t@  2 root  admin    68 10 15 09:17 cores
dr-xr-xr-x   3 root  wheel  4241  1 11 09:17 dev
lrwxr-xr-x@  1 root  wheel    11 10 15 09:16 etc -> private/etc
dr-xr-xr-x   2 root  wheel     1  1 11 09:18 home
-rw-r--r--@  1 root  wheel   313  8 23 10:35 installer.failurerequests
dr-xr-xr-x   2 root  wheel     1  1 11 09:18 net
drwxr-xr-x   3 root  wheel   102 10 29 20:57 opt
drwxr-xr-x@  6 root  wheel   204 10 15 09:17 private
drwxr-xr-x@ 59 root  wheel  2006 12  7 16:28 sbin
lrwxr-xr-x@  1 root  wheel    11 10 15 09:16 tmp -> private/tmp
drwxr-xr-x@ 12 root  wheel   408 10 15 09:25 usr
lrwxr-xr-x@  1 root  wheel    11 10 15 09:17 var -> private/var
```

* Ubutu文件系统

```
drwxr-xr-x   2 root root  4096 Mar 25  2015 bin
drwxr-xr-x   3 root root  4096 Mar 25  2015 boot
drwxr-xr-x  13 root root  3820 Nov 30 06:50 dev
drwxr-xr-x  89 root root  4096 Jan  7 01:35 etc
drwxr-xr-x   3 root root  4096 Nov 30 06:50 home
lrwxrwxrwx   1 root root    33 Mar 25  2015 initrd.img -> boot/initrd.img-3.13.0-48-generic
drwxr-xr-x  21 root root  4096 Nov 30 07:05 lib
drwxr-xr-x   2 root root  4096 Mar 25  2015 lib64
drwx------   2 root root 16384 Mar 25  2015 lost+found
drwxr-xr-x   2 root root  4096 Mar 25  2015 media
drwxr-xr-x   2 root root  4096 Apr 10  2014 mnt
drwxr-xr-x   2 root root  4096 Mar 25  2015 opt
dr-xr-xr-x 114 root root     0 Nov 30 06:50 proc
drwx------   3 root root  4096 Nov 30 06:50 root
drwxr-xr-x  18 root root   680 Jan 13 06:29 run
drwxr-xr-x   2 root root  4096 Mar 25  2015 sbin
drwxr-xr-x   2 root root  4096 Mar 25  2015 srv
dr-xr-xr-x  13 root root     0 Nov 30 06:50 sys
drwxrwxrwt   2 root root  4096 Jan 13 06:25 tmp
drwxr-xr-x  10 root root  4096 Mar 25  2015 usr
drwxr-xr-x  12 root root  4096 Mar 25  2015 var
lrwxrwxrwx   1 root root    30 Mar 25  2015 vmlinuz -> boot/vmlinuz-3.13.0-48-generic
```

* iOS文件系统

```
lrwxr-xr-x  1 root admin     32 Dec 31 17:50 Applications -> /var/stash/_.jQJG6s/Applications
drwxrwxr-x  3 root admin    102 Dec 31 17:46 Developer
drwxrwxr-x 17 root admin    782 Dec 31 21:59 Library
drwxr-xr-x  3 root wheel    102 Aug 14  2013 System
lrwxr-xr-x  1 root admin     11 Jan 10 19:21 User -> /var/mobile
drwxr-xr-x  2 root wheel   2006 Dec 31 17:46 bin
drwxr-xr-x  2 root wheel     68 Oct 28  2006 boot
drwxrwxr-t  2 root admin     68 Aug 14  2013 cores
dr-xr-xr-x  3 root wheel   1236 Jan 10 19:21 dev
lrwxr-xr-x  1 root admin     11 Nov  8  2013 etc -> private/etc
drwxr-xr-x  2 root wheel     68 Oct 28  2006 lib
drwxr-xr-x  2 root wheel     68 Oct 28  2006 mnt
-rwxr-xr-x  1 root admin 521264 Dec 31 17:46 pguntether
drwxr-xr-x  4 root wheel    136 Jan 10  2014 private
drwxr-xr-x  2 root wheel    612 Dec 31 17:46 sbin
lrwxr-xr-x  1 root admin     15 Nov  8  2013 tmp -> private/var/tmp
drwxr-xr-x  8 root wheel    340 Dec 31 17:51 usr
lrwxr-xr-x  1 root admin     11 Nov  8  2013 var -> private/var
```



2. OS X优点：GUI，Shell（UNIX），触控板

	* Shell：是Linux/Unix系统的一个外壳，可以理解成衣服，负责外界与Linux内核的交互，接受用户或其他应用程序的命令，然后把这些命令转化成内核能理解的语言，内核是真正干活的，干完活以后返回结果

	* 关于触控板，默认没有打开的选项：1. 双指轻拍代替右键 2. 触发角，右下角回到桌面 3. 三指滑动选择

3. 软件推荐：（几乎每天都会使用的，带来无数便捷，节省无数时间的）

	* Evernote：知识点收集工具，配合`印象笔记-剪藏`Chrome插件使用
	* Alfred：Mac上的神兵利器 [安装使用指南](http://macshuo.com/?p=625)
	* Iterm2：Mac终端替换方案 [安装](https://www.iterm2.com/)

	* Zsh：Shell替换方案[安装使用指南](http://macshuo.com/?p=676)
	* Charles：网络抓包工具 [使用指南](http://blog.devtang.com/blog/2015/11/14/charles-introduction/)
	* Reveal：UI调试工具 [使用指南](http://security.ios-wiki.com/issue-3-4/)


4. iOS优点：系统升级率高，系统稳定、流畅，安全？，技术领先（TouchID，3DTouch）

#### 如何学好一门编程语言

1. 明确的目的（为什么学习、达到什么程度、花费多长时间）

2. 经典教程

	* C语言程序设计
	* iOS人机交互指南[链接](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/Controls.html#//apple_ref/doc/uid/TP40006556-CH15-SW1)
	* 苹果官方Library[链接](https://developer.apple.com/library/ios/navigation/)
	* 从零基础到App Store上架[链接](https://book.douban.com/subject/26372385/)
	* MacTalk 人生元编程 [链接](https://book.douban.com/subject/25826578/)


3. 掌握基础，持续练习（每天花费数小时）
4. Master （找到已经掌握这个语言的人指导）
5. 社区
	* 技术问答社区：[stack over flow](http://stackoverflow.com/)
	* 开源项目社区：[github](https://github.com/)
	* Google: 关于google搜索技巧：
	* 优秀的个人博客（待增加）
	* 优秀的公众号（待增加）
6. 学习环境（逃离舒适区）找到组织，这个也是我成立这个iOS学习沙龙的初衷之一


### App沙盒目录、Bundle目录介绍

文件系统介绍：[点击](https://developer.apple.com/library/ios/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html)

一图胜前言：

![](http://photo-coder.b0.upaiyun.com/img/how-to-learn-ios.png)

#### 沙盒


```
	|- Documents
	|
	|- Library
	|   |-- Caches
	|   |-- Preferences 
	|   |   |---- com.xxx.plist
	|- tmp
		
```
* Documents -- 保存用户产生的内容，只保存希望暴漏用户的文件，iTunes会备份这个文件夹下的内容 
* Library -- 保存非用户产生的内容，保存不想暴漏给用户的文件， iTunes会备份这个文件夹下的内容
* tmp -- 缓存文件夹，缓存网络图片等，iTunes会备份这个文件夹下的内容。


#### Bundle

```
	|- $(EXECUTABLE_NAME)
	|
	|- Base.lproj
	|   |-- LaunchScreen.storyboardc
	|   |-- Main.storyboardc 
	|   
	|- Info.plist
```
Bundle包含app及其资源文件，不可写，具有签名保护，一旦破坏可能无法启动。


### Xcode、iOS Project简介

#### 项目目录介绍

```
AppDelegate 
Models 
Macro 
General 
Helpers 
Vendors 
Sections 
Resources
```

#### Target（bundle id，arc，code sign，bitcode，prefixfile）
