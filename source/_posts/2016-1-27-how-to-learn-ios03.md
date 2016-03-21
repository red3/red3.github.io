
---
layout: post
title:  "iOS学习沙龙系列（三）：ContentView以及ContainerViewController"
date:   2016-1-27 10:05:00
---



# 前言

## 书籍推荐

关于时间管理的思考：按照目前正常的进度，iOS学习沙龙已经进行到第三讲，意味着这个沙龙开课已经过去二周了，按照每天3小时的编程工作量，也应该有花费差不多42个小时在iOS编程上面，也应该差不多了解到了Objective-C这门语言的基础语法，Foundation框架的使用，UIKit框架的使用。

当然这是正常的按照计划的情况，事实上，几乎所有的规划到头来都不是按部就班执行过来的，我自己的经验也是这样，本来要求自己今天必须完成某件事情，实际做起来总想先看个电影，再小睡一会，结果一天过去了发现任务还没有完成。

回过头来其实发现我们都犯了一个严重的错误：对短期的收益要求太高，对长期收益要求太低。对收益的把握不平衡导致了各种各种的时间管理的问题。

但是，这真的是时间管理出了问题了吗？

李笑来老师在《把时间当做朋友》中这本书中说道：时间是不可管理的，真正需要管理的是人才对。我从读了这本书以后，也慢慢开始学习管理自己。

[在线阅读地址](http://zhibimo.com/books/xiaolai/ba-shi-jian-dang-zuo-peng-you)

[京东链接](http://item.jd.com/11338691.html)



# 本期纲要

## ContentView

### UIScrollView

当需要显示的内容区域要大于View的边界的时候，需要用UIScrollView来滑动显示不可见的范围至可见的范围。设备的大小是固定的，所以一个屏幕上显示的内容大小也被限制，当屏幕上需要显示将A、B、C、D、E、F等不同View按垂直方向排列显示的时候，就需要用到UIScrollView来显示内容

* ContentSize : 内容区域的大小，通过这个属性，ScrollView得知自己需不需要滑动来呈现屏幕外的内容。
* 速度（decelerationRate）：滑动的速度
* 方向 ：可水平方向滑动，可垂直方向滑动
* Pagemode ： 一次滑动一屏，经常用于广告banner的滑动模式

### UITableView

单列多行的列表。继承自UIScrollView，当要显示的内容具有规律性、重复性，或者说就像列表中的行的View是一致的，需要用到UITableView来显示内容

* 复用机制，当列表行数足够大，意味着TableView上需要显示的行数有很多，怎么保证内存开销（`ReuseIdentifier`）？
* 数据或者行，可以被分割成多组，或者group
* 提供删除、增加行，移动，选择（多选）
* AccessoryType（EditingAccessoryType）: 列表中每一行Cell的辅助指示标志，最常见的为开口向右的小箭头。


### UICollectionView

跟UITableView及其相像，继承自UIScrollView，在展示形式上有着更高级的方式。

### UITextView

显示大段文字的ScrollView。

### UIWebView
也是UIScrollView，用来加载网页。

## Bar

### UIStatusBars
状态栏，位于屏幕最顶部，透明，呈现关于设备的信息。

* 轻触以回到顶部
* 网络活动指示
* Dark、Light两种Style
* 如有必要，全屏浏览多媒体信息时需要隐藏状态栏

### UINavigationBar

导航栏，位于StatusBar底部，透明，通过呈现层级关系信息来导航，提供对内容的管理。

> 包含在UINabigationController中


通过NavigationBar来提供不同View场景之间的切换的导航功能，适当的时候，还需要提供一个UIBarButtonItem来对当下的View的内容进行管理，比如修好了改价页面导航条右边的编辑按钮，比如短信界面导航栏左面的编辑按钮。

* 当导航到下一导航层级页面的时候，有两件事请发生：
	1. 设置当下任务（或者说当下View）的title，这个title在NavigationBar中间显示
	2. 设置返回按钮（backButton），可以的话还需要指示上一层级的title

* 导航上应该尽可能只有三样东西：title、backButton、提供编辑功能的item
* 可Tint，（tintColor属性），比如说，当把UINavigationBar的tintColor设置为绿色，它所管理的视图，比如有文字显示的View，也被着色为绿色
* 返回按钮（backButton） 的箭头图片可定制，但要在保证用户可辨认的前提下定制。
* 如有必要，全屏浏览多媒体信息时需要隐藏状态栏


### UIToolBar

对于用户交互按钮的一组集合，出现在屏幕底部。邮件app的底部有用到，国内的开发者好像不太喜欢使用这个控件，大致了解即可。

### UITabBar

tab栏，提供用户在不同栏目（View）之间切换的功能，透明的，屏幕底部，AppStore中绝大部分应用用到UITabBar来管理app中的各个栏目。

> 包含在UITabBarController中

使用UITabBarController来作为app的主View，因为TabBar会将app的主要栏目都呈现在自己的视图上，用户只要扫一眼TabBar就知道了一个app的组成以及功能部分，也很容易知道自己应该去往哪个栏目。

* Badge ：考虑使用角标来向用户传达显著的消息通知。


### UIBarButtonItem && UITabBarItem

UIBarButtonItem是被UINavigationBar、UIToolBar管理的对象，上文中提到的导航条上的Item即为UIBarButtonItem，提供交互行为。
UITabBarItem是被UITabBar管理的对象。

## Container View Controller

#### UIViewController

app中扮演至关重要的角色，简单来说，可以将当下屏幕中显示的内容（场景）比作一个ViewController，因此，一个app至少有一个ViewController。场景的切换就是ViewController的切换，当然，按照面向对象的思考方式，必然有一个对象是设计出来要管理这些场景的切换（UINavigationController、UITabBarController就是这样的存在）。

具体到单一的某个ViewController，它的职责是：

* 提供一个画布（属性`view`），来管理SubViews

```objective-c
[self.view addSubView:...]

```
* 处理UI交互（方法）

```objective-c
[self.button addTarget:<#(nullable id)#> action:<#(nonnull SEL)#> forControlEvents:<#(UIControlEvents)#>]

```
* 两种方式来将一个UIViewController显示在屏幕上
	1. 交由ContainerViewController来管理 （push、pop）
	2. 模态推出(present)一个视图控制器
	
	```objective-c
     UINavigationController *naviVC = [[UINavigationController alloc] initWithRootViewController:aVC];
    [self presentViewController:naviVC animated:YES completion:NULL];

	```

图示以及真相：

![](http://photo-coder.b0.upaiyun.com/img/how_to_learn_ios3_01.png)

详细的说明，以及关于怎么在StoryBoard中连接View到ViewController中，请参考这个[链接](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/index.html)



#### UINavigationViewController

用来管理ViewController的容器，提供导航，即不同ViewController之间的转场切换。

从组分上来讲
* 容器属性（属性viewControllers）：包含着当前任务栈中所有的ViewController，任务栈最上面的ViewController即为当前可见的ViewController
* NavigationBar：Bar的整体表现（比如backgroundColor，tintColor）可以自定义，但有一点需要说明，当下场景中可见的Bar上的内容，我们称之为navigationItem，是由当下任务栈的ViewController来提供的，理由很简单，作业场景都换了，没有理由上面的导航信息不更换。

从功能来讲
* Push ： 显示下一等级或者步骤的页面

```objective-c
   [self.navigationController pushViewController:aVC animated:YES];

```
* Pop ： 回到上一等级或者步骤的页面

```objective-c
   [self.navigationController popViewController:aVC animated:YES];

```


图示以及真相：

![](http://photo-coder.b0.upaiyun.com/img/how_to_learn_ios3_02.png)

详细的说明，以及关于怎么在StoryBoard中拖拽导航的子视图场景切换，请参考这个[链接](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/NavigationControllers.html)


#### UITabBarViewController

用来管理ViewController的容器，提供app的栏目之间的切换。

从组分上来讲
* 容器属性（属性viewControllers）：一般来说，应该包含app的几个主要栏目，每个栏目可能是UINavigationViewController，也可以是UIViewController
* TabBar：Bar的整体表现（比如backgroundColor，tintColor）可以自定义，但有一点需要说明，在Bar需要显示之前，Bar上的可见内容，我们称之为tabBarItem，是由UITabBarController管理的各个栏目的ViewController来提供的。理由也很简单，这个tabBarItem是用来指示这个栏目的，当然由这个栏目来负责创建。

从功能上讲
* SelectedIndex：当下选中的栏目的下标，通过这个下标，可以控制app的栏目之间的切换。

图示以及真相：

![](http://photo-coder.b0.upaiyun.com/img/how_to_learn_ios3_03.jpg)


详细的说明，请参考这个[链接](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewControllerCatalog/Chapters/TabBarControllers.html)

#### UIWindow以及上面所讲的之间的联系

先看这张图：

![](http://photo-coder.b0.upaiyun.com/img/how_to_learn_ios3_04.png)

从图中可以看到，UIWindow是所有内容的承载，window通过rootViewController来将生产内容的职责赋予给Controller，这个Controller一般来讲是UITabBarController，也可以是UINavigationController。广义上来说是ContainerViewController，确定了这个ContainerViewController以后，Container管理的subViewController通过自己的 `navigationItem` 与 `tabBarItem` 和 自己的容器之间交流信息。


## 附件下载

* [附件01](https://redharry.b0.upaiyun.com/download/UIKitDemo02.zip): UIScrollView、UITableView以及ContainerViewController


## 下次课程概要

UIViewController的生命周期、内存管理













