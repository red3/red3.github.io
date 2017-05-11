---
layout: post
title:  "如何才能使得 Charles 和 Shadowsocks 共存"
date:   2016-11-1 18:30:00
---

## 引子

不知道诸位程序员君有没有在接口抓包过程中发现抓包程序中毛都没有，然后搜索发现原来是跟翻墙程序有冲突，于是乎往后每次需要抓包都需要事先关闭翻墙程序，然后抓包完毕再将翻墙程序打开。。。

长此以往，叔能忍，婶婶不能忍啊！

这篇文章的主要作用就是解决大家的这个烦恼。

## 剖析原理

想要解决这个冲突问题，我们就要先从它们的原理入手，看冲突的根源是什么，方可对症下药。

我们就要抓包软件和翻墙软件中的佼佼者来说，抓包软件我们选择 [Charles](https://www.charlesproxy.com/), 翻墙软件我们选择 [Shadowsocks](https://github.com/shadowsocks). 

抓包也好，翻墙也好，其实都是通过修改系统的网络服务的代理配置来完成的，我们自己也是可以看到这个设置的，通过路径 **System Preferences -> Network -> Advanced -> Proxies** 便可一窥究竟。

这里面，Shadowsocks 是通过设置 **Automatic Proxy Configuration** 指定一个配置文件的方式来完成代理设置的，之所以这样，是因为 Shadowsocks 有两种工作模式，即自动模式和全局模式，这两种模式的切换，实际上就是通过修改这个配置文件来完成的。

而 Charles 是通过设置 **Web Proxy(HTTP) 和 Secure Web Proxy(HTTPS)** 来完成代理设置的，具体来说，就是将本机的 HTTP 流量和 HTTPS 流量都转发到了 8888 端口上了（这个跟 Charles 里面的设置是对应的）。

通过以上剖析我们发现，这二者其实都是通过相同的办法来使代理生效的，故不可共存！并且更为坑爹的是，当你打开了 Charles 抓包的时候，发现 Shadowsocks 还在运行着，这个时候你肯定会想，先把 Shadowsocks 临时关闭下吧，然后你就会发现这个玩意不光把自己的配置给关闭了，还捎带着把 Charles 的配置也给关闭了，于是你不得不退出 Charles 一次，然后再次打开以使配置重新生效，就说坑不坑吧！

这绝对是逼我们动真格的了。

## 解决方案

那么到底有没有办法让这两个东西共存呢？

答案就是：绝对没有！☺️

原谅我标题党了，大家先别忙着拍我，因为原理大家都看到了，理论上来说，确实没有办法共存，但是在这种情况下，我们就需要动用我们人人都可以当产品经理的产品思维来思考下，做个思维转换：

因为 Shadowsocks 的关闭导致了 Charles 的配置也被修改，也就是说 Shadowsocks 关闭了不该关闭的东西，那么我们是否可以跳过 Shadowsocks 的程序控制，自己去手动控制呢？

答案当然是可以的，不信当你打开 Charles 的时候，再去系统网络代理设置里面把 Shadowsocks 的 **Automatic Proxy Configuration** 关闭，看看是否就可以正常抓包了？

可惜老是手动去配置怎么看也不是很符合我们的产品经理思维啊！：）


那么就要隆重推荐今天的主角了：[proxy-switcher-alfred-workflow](https://github.com/lululau/proxy-switcher-alfred-workflow) (以下简称 proxy-switcher).


Proxy-switcher 是一款 Alfred 的 workflow 插件，通过这个插件，这个可以快速通过 Aflred 的 Toggle 窗口来切换网络代理控制的开关。

千言万语不如上张图啊，我们来看看效果：

![](https://photo-coder.b0.upaiyun.com/img/ss-charles01.gif)

怎么样，很爽吧，以后再有抓包的时候，我们直接呼出 Alfred 键入我们的快键键临时关闭 Shadowsocks 的代理设置，等抓包完毕，我们再使用同样的方法打开就是了。

心动了吗，赶快去这个插件的项目主页下载安装试试吧。


