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
当然，首先我们要有一个iOS app，因为WatchKit app的bundle是集成与iOS app中的，安装Watch app的时候会把WatchKit app的bundle从iPhone中传输到Watch上。而对于Watch app来说，分为两个部分：WatchKit app(运行在Apple Watch)以及WatchKit extension(运行在iPhone上)。WatchKit app只包含程序UI相关的storyboards和资源文件（比如图片等），而WatchKit extension包含管理和响应UI的代码。
#添加Watch app Target

和添加一个iOS app target一样的步骤，如果我要添加了一个名为“Watch”的Target，顺序是下面这样的：
![](/assets/2015/img-watchdemo-1.png)

选择Next以后，需要勾选这些东西：
![](/assets/2015/img-watchdemo-2.png)
分别解释下这几个选项

- Notification Scene 通知中心：应用程序收到远程或本地通知的时候，手表界面出现的通知布景。
- Glacne 速览。从表盘界面上拉出现的界面，添加到速览列表后，用户可以通过左右滑动速览界面，看到程序提供的速览界面，提供最重要的信息，不可滚动的，一屏，不可包含可交互元素，点击会启动WatchKit app
- Complication 特殊功能（或者叫功能栏）：抬起手腕可以在表盘上看见的东西，比如我的表盘上自定义的功能栏有日历、健身活动、定时器等。

这些按需要勾选，确定以后，在WatchKit app的storyboard中就可以看到以下界面：
![](/assets/2015/img-watchdemo-3.png)
图中的Main interface就是Watchkit app的主界面了，Glance Interface是速览界面，剩下的Static Interface和Dynamic Interface是连着一起的，可以这样解释：当Watch上收到iPhone的通知的时候，首先会调起程序对应的Static Interface，当用户对通知表示有兴趣，也就是说抬起了手腕想看这条通知的时候，会切换到对应的Dynamic Interface上。

此次，我们的任务就是把这几个Interface都运行起来，在Watch上看到效果。
#添加界面元素
暂时，我们可以先在各个界面上拖上一些简单的元素。这里需要注意的是Glance Interface上是不允许可交互元素（比如button，switch等）的存在的。
#编译运行
到现在为止，查看Scheme menu可以看到，Xcode已经为我们生成了调试不同界面的Scheme了。要编译运行main Interface我们选中Watch Scheme，就可以在模拟器上看到我们的main Interface了。
Glance和Notification Interface稍微还要费电周折，分别讲一下：

- Glance

要选择iOS app的target对应的Scheme，编译运行，在iPhone的“Watch”应用中，进入我们新建的iOS app设置界面，选中“Show In Glance”（在速览中显示）。然后回到Watch模拟器，从表盘界面往上滑动，再往右滑动，就可以看到我们创建的速览界面了。

- Notification Scene

在工程的“Edit Scheme”中，选择编辑“Notification-Watch”:
![]( /assets/2015/img-watchdemo-4.png)
有两个地方需要注意：

1. Watch Interface 可以切换Static Notification和Dynamic Notification来分别调试不同场景下的通知视图。当然可以通过左下角的Duplicate Scheme来复制一个Scheme，这样就分别有了Static Notification和Dynamic Notification对应的Scheme了。
2. Notification Payload 在Watch Extension文件夹下的Support Fils下有一个名为“PushNotificationPayload.apns”的文件。这个文件是Xcode为我们自动生成的，因为模拟器不支持推送，所以通过这个文件模拟一个推送。我们可以修改这个文件的内容，以及创建新的payload文件，然后再来调试我们的程序。

现在我们选中Static Notification的Scheme来编译运行，就可以看到模拟器上收到的通知内容为“Test Message”，而这个文字是从PushNotificationPayload.apns这个文件中读写出来的。

然后再选中Dynamic Notification的Scheme来编译运行，发现怎么还是刚才的通知，现在我们要移步到NotificationController.m这个文件下，将一下代码的注释打开



{% highlight Objective-C %}
- (void)didReceiveRemoteNotification:(NSDictionary *)remoteNotification withCompletion:(void (^)(WKUserNotificationInterfaceType))completionHandler {
    // This method is called when a remote notification needs to be presented.
    // Implement it if you use a dynamic notification interface.
    // Populate your dynamic notification interface as quickly as possible.
    //
    // After populating your dynamic notification interface call the completion block.
    completionHandler(WKUserNotificationInterfaceTypeDefault);
}
{% endhighlight %}

然后我们需要把下面这行代码中的"WKUserNotificationInterfaceTypeDefault"替换为"WKUserNotificationInterfaceTypeCustom"，然后重新编译运行，就发现我们定制的Dynamic Notification出现在屏幕上了。
	    			
	completionHandler(WKUserNotificationInterfaceTypeDefault);


#下期预告
今天的系列就先到这里，下一系列讲讲怎么创建一个功能栏，就是抬腕可以在表盘上看见的那个东西。

- 惯例：[点击这里下载Demo](https://github.com/red3/WatchDemo/archive/master.zip)
- 重要：因为博客暂时没有评论区，如果你对本文有什么疑惑，或者你发现本文由存在错误的地方，非常欢迎你联系我指出。QQ：309333018，WeiChat：flipgo








