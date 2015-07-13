---
layout: post
title:  "为什么我们需要Present这种模态推出视图的方式？"
date:   2015-07-11 10:37:36
categories: jekyll update
---
iOS开发者都知道，推出新页面有两种方式，一种是导航Push，另一种是视图本身Present另一个视图。一般的应用程序都是UITabBarController+4个UINavigationController，然后每个导航Push或者Pop就解决了问题。

这次引发我思考的是佳缘经纪人项目中要添加银行卡的一个功能：这个功能有两个入口，第一个入口是从导航栈中的某个视图Push出的（非根视图），第二个入口是从导航的根视图Push出的。这个功能本身又需要分2个步骤，验证手机收到的验证码以及输入银行卡号，是两个页面。这样在成功添加银行卡号后要跳转到发起该请求的页面的时候可以这样：
- 如果是根视图：PopToRootViewController
- 如果不是根视图：先找到导航栈中的那个控制器，然后PopToViewController：
从流程上讲：它应该是这个样子

![](/assets/2015/broker_add_bankCard.png)

那么问题来了，如果哪天这个功能还要有一个入口，在Pop回去的地方就又要多加判断，意味着这个流程里面就有了很多垃圾代码去判断是从哪个页面来的。这个时候我开始考虑，这样的设计可能是存在问题的。

###微信

意识到这个问题的时候，我在想，有类似流程设计的肯定不止一家，其实是可以参考其他软件是怎么设计的。于是花费了时间，看了看微信在类似的场景的设计。下图是微信的更换绑定手机号的流程截图，其实“更换手机号”这个页面是“绑定手机号”这个页面Present出的。

![](/assets/2015/wechat_change_phone.png)

###苹果在iOS框架中的设计

其实苹果自己设计的UIImagePickerController就是一个值得参考的例子。
它的流程是这样的

![](/assets/2015/ios_imagePicker.png)
“Photos”这个页面是被Present出的，并且在Xcode中可以看到UIImagePickerController是继承自UINavigationController的。

###因此

可以看出，iOS框架和微信App中，在做这样的操作的时候（这个操作一般可以表述为：需要用户去输入，去选择，去编辑。这个操作一般还涉及好几个步骤，好几个页面），一般用Present的方式模态出一个导航栈出来。而这样做的具体优点为：

- 流程中，用户想放弃编辑，通过导航中的“取消”，可以随时Dismiss这个导航栈，然后回到发起这个功能的页面。而相比直接用导航Push这一系列流程的时候，用户想放弃编辑，只能通过导航中的“返回”一步一步返回了。
- 用户完成操作的自动跳转，可以通过Dismiss直接回到页面发起的地方。而相比直接用导航Push这一系列流程的方式，要写一堆代码判断到底应该Pop回哪个页面。

所以参照这些，我把红娘经纪人中添加银行卡的步骤设计成类似的流程。同样的，我们添加银行卡的这个流程也有这样一个协议，用来表明是添加银行卡成功了还是取消了这次添加。

	@protocol UIImagePickerControllerDelegate<NSObject>
	@optional
	// The picker does not dismiss itself; the client dismisses it in these callbacks.
	// The delegate will receive one or the other, but not both, depending whether the user
	// confirms or cancels.
	- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info;
	- (void)imagePickerControllerDidCancel:(UIImagePickerController *)picker;
	@end

现在回头看微信更换绑定手机的设计，其实也是有点瑕疵的：在“填写验证码”这个环节，缺少“取消”的按钮，如果用户想放弃这次更换手机号的操作，只能先返回到上级界面再点击“取消”了。

###最后说一点

最近读郝培强（tiny4voice）微信公众号中的一篇文章很有感悟，文章最后说：

	我大家的自主努力，不是坚持，不是在泥潭里坚持，
	而是积累，积累改进，积累思考，要跳出泥潭，掌控工作，掌控生活，掌控自己
	
所以我想我们在开发中不要总是一味去照搬以前的方式去蛮干，而要思考，有没有更好的方式。经常的，还要回过头来看看自己写的代码，到底设计得是不是够好，够规范，够易维护。这样才能有所积累。



