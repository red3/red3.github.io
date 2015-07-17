---
layout: post
title:  "Apple Watch开发系列（一）"
date:   2015-07-16 22:52:36
categories: jekyll update
tags:
  - Apple Watch
  - WWDC
---

本系列是我依据WWDC15 Session Video以及苹果官方文档写的一个小Demo，涉及的资料有

-[Building Watch Apps](https://developer.apple.com/videos/wwdc/2015/?id=108)

-[AppleWatch2 Transition Guide](https://developer.apple.com/library/prerelease/watchos/documentation/General/Conceptual/AppleWatch2TransitionGuide/index.html)
-[WatchKit Programming Guide](https://developer.apple.com/library/prerelease/watchos/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html)

另外需要说明的是本Demo是基于Watch OS 2.0开发的，因为我开始学的时候已经是2.0了，哈哈。在该系列的每篇文章末尾会有一个链接地址，指向这个Demo。

#前言
在开始之前，确保我们有一个iOS App，因为Watch App的bundle是集成与iOS App中的，安装Watch App的时候会把Watch App的bundle从iPhone中传输到Watch上。而对于Watch App来说，分为两个部分：WatchKit app(运行在Apple Watch)以及WatchKit extension(运行在iPhone上)。WatchKit app只包含程序UI相关的storyboards和资源文件（比如图片等），而WatchKit extension包含管理和响应UI的代码。

- Glacne 速览。从表盘界面上拉出现的界面，添加到速览列表后，用户可以通过左右滑动速览界面，看到程序提供的速览界面，提供最重要的信息，不可滚动的，一屏，不可包含可交互元素，点击会启动WatchKit app
- Notification
- 






