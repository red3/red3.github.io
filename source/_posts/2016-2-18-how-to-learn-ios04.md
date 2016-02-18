# iOS Share课程（四）

## 学习路线回顾


## 本期纲要

本期沙龙由于春节放假原因，部分同事还没有上班，故不进行新的内容讲解，着重解决下学习iOS过程中遇到的问题。

## 问题

1. `AppDelegate`这个文件和`UIApplication`的关系以及这个文件中能写哪些代码？

`UIApplication`比作手机中的一个我们编写的app，`AppDelegate`这个文件（包括.h和.m文件）就是这个app的代理。每一个app都需要一个代理来响应app的状态变化。举例来说，当app首次启动，进入后台，或者再次进入前台的时刻就会通过代理（也就是`AppDelegate`这个文件）来下发这些状态的变化。远程的消息通知，app之间的信息交换，也是通过这个文件来下发的。Xcode再每次创建新工程的时候其实会自动生成这个代理文件，并把这个代理赋值给app，不需要开发人员去额外操作。

这个文件中应该写哪些代码：AppDelegate作为程序级状态变化的delegate，应该只做`路由`和`分发`的作用，具体逻辑实现代码还是应该在分别的模块中，这个文件应该保持整洁，除了<UIApplicationDelegate>的方法外不应该出现其他方法。


2. ViewController中的`view`属性跟XIB的关系？
3. 如何在storyboard中编辑UI

4. XIB中的UI控件应该链接在ViewController的哪个地方？

Xcode中，.h文件是接口文件，.m是实现文件

5. XIB中的UITextField怎么绑定事件方法？
6. UITableView的工作原理，UITableview中的Cell怎么配置不同高度？

这个比较复杂，下节课会重点讲讲这个。

7. 如何在Xcode中新建不同类型的文件？

首先定位到导航区的工程选项卡下面需要新建文件的目标文件夹->`File-New-File` 或者`Command+N`


	1. Source:

		* Cocoa Touch Class: 最常用的选择，新建ViewController的子类，或者新建UIView的子类等都从这里开始
		* Objective-C File: 可以创建category, protocol文件。
		* Head File: 单独创建一个头文件。比如可以单独创建一个头文件来生命一些通知常量

	2. User Interface:

		* Storyboard: 故事版，在这个文件中编辑UI。多人协作建议创建多个Storyboard文件，避免同时编辑一个Storyboard文件带来版本冲突的问题
		* View: XIB文件，带一个空白的UIView
		* Launch Screen：iOS8.0以后推荐使用Launch Screen这个XIB文件来制作启动图

	3. Resource

		* Property List: plist文件。Xcode中用于序列化文本的文件。可以以数组或者字典的形式存储数据。典型的用法有使用plist文件来持久化全国城市列表。
		* Strings File: 字符串文件，用于国际化字符串

	4. Other

		* PCH File: prefix header file, 即预编译文件，在这个文件中导入的头文件，整个工程都可用，其他地方不需重复导入。当然不建议在这个文件中大量导入头文件，应该只导入那些工程中频繁使用的文件

8. 如何Debug

详细的Debug教程参考官方的这个[教程](https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/debugging_with_xcode/chapters/about_debugging_w_xcode.html)
进阶的教程参考这个[链接](http://objccn.io/issue-19-2/)


9. 怎么理解ViewController的生命周期

这个会在下节课重点讲，这节课先大致给出点说明，有个大致的了解。

首先说明一下，在运行时ViewController是怎么动态获取到Storyboard中关于UI的信息的以及跟自身关联起来的

1. 通过Storyboard或者XIB中的配置信息实例化对应的UI控件
2. 连接绑定IBOutlet 和 IBAction
3. 将Storyboard中的rootView赋值给Viewcontroller的`view`属性
4. 调用ViewController的`awakeFromNib`方法
5. 调用ViewController的`viewDidLoad`方法

具体的ViewController之间的转场中，对应的view的生命周期：

``` objective-c

- (void)loadView;

不用主动重写这个方法。view这个property被使用，且这个property值为空，这个方法就被调用，为的是创建一个view并把它赋值给vc的view属性。
有nib时，从nib加载，没有nib时，这个方法创建一个UIView对象。

如果用nib，永远不要重写这个方法。
想自定义这个view对象，可以重写，但是不要call super

- (void)viewDidLoad;

一般需要重写这个方法，来add or remove子view或者modified从nib中加载来的子view的状态，load data网络请求

- (void)viewWillAppear:(BOOL)animated

改变状态栏，导航栏 must call super

- (void)viewDidAppear:(BOOL)animated

views are on screen. must call super

- (void)viewWillDisappear:(BOOL)animated

改变状态栏，导航栏, must call super

- (void)viewDidDisappear:(BOOL)animated

views are not on screen. must call super

```



## 附件下载

附件01：[roadMapIOS](everythingcomputerscience.com/books/RoadMapiOS.pdf)


## 下次课程概要

内存管理













