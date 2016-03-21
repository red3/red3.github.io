---
layout: post
title:  "iOS学习沙龙系列（六）：iOS中常见的设计模式"
date:   2016-3-2 20:28:00
---

## iOS中常见的设计模式

这次课主要讲解iOS中常见的设计模式。

我们经常听到古装片上有“以文会友”或者书生们常说的“以文会友”，那么对于程序员来说，我们就是“以代码会友”，那当我们拿代码会友的时候，就要讲究我们的招式，设计模式其实就是代码的招式。关于设计模式，更简单来讲，就是为了让代码更简单，让工程更易维护。 

iOS中常见的设计模式有MVC、代理、单例、KVO、命令（target-action），工厂方法，类簇等。我们主要挑几个常见的作说明。



### MVC

MVC是一种古老的软件设计模式了，用来实现跟用户界面相关的工程中。iOS中也没有例外采用了这种方式，这也是苹果开发指南之极力推崇的开发方式。关于MVC中各自代表的意义及作用，如下：

* M是Model，包含数据和基本的行为，包含应用程序所需的数据，以及定义了操作数据的基本逻辑。

* V是View，负责呈现信息，View知道如何显示信息，并且允许用户交互，View不应该对保存界面上的信息负责。

* C是Controller，负责联系Model和View。Controller是view和model之间的中间人。controller需要将model中保存的数据传递到view以供显示，同时还需要响应view上的变化，将变化传递到model以供存储。

文章末尾的附件Demo，依据MVC的分工职责作了划分，可以参考一下。

### Delegate

代理是当某个对象发生的某些事件时的代表者。
需要代理的对象经常是具有响应能力的对象，比如UIApplication、UITableview、UITextField等。而代理者通常是管理这个响应对象的对象，比如ViewController。

为什么要使用代理这种设计模式？我们先看看Cocoa中在什么情形下使用代理

下面的方法摘自`UITextFieldDelegate`中：

	- (BOOL)textFieldShouldBeginEditing:(UITextField *)textField;        
	- (void)textFieldDidBeginEditing:(UITextField *)textField;           
	- (BOOL)textFieldShouldEndEditing:(UITextField *)textField;

可以看到UITextfield在响应用户交互的时候，有众多状态发生，也附带众多事件，而这里面的某些函数的返回值又需要依据具体的业务来判断（比如在输入手机号的时候判断位数达到要求后就不再响应输入），textfield作为视图控件是不应该事先知晓业务逻辑的，所以UITextfield类的任何使用者都可以仅仅实现代理协议UITextfieldDelegate，而不用继承该类就能完成业务逻辑，而UITextfield类也能避免无关的逻辑而能够保持其通用性。

### 单例

单例是作为程序中共享状态的一种设计，经常认为是创建一次，永久有效。或者是永远只有一个实例。

单例是整个Cocoa中被广泛使用的核心设计模式之一。并且苹果开发者库把单例作为“Cocoa 核心竞争力”之一。iOS中常见的单例有`UIApplication` 、 `NSFileManager` 和 `NSUserDefault`等。

从Xcode中内建有创建单例的代码片段（"通过`dispatch_once`关键字激活"）中确可以看出单例在Cocoa中的重要性:

``` 
+ (instancetype)sharedInstance {
	static dispatch_once_t once;
	staticid sharedInstance;
	dispatch_once(&once, ^{ 
		sharedInstance = [[self alloc] init]; 
	});
	return sharedInstance;
}
```

当程序中的某个服务只有一个实例，可以考虑设计成单例。类似的情况比如网络服务，或者地理位置服务等。

但是应该注意的是单例要用在合适的地方，避免滥用单例。推荐阅读这篇文章：[被滥用的单例](http://objccn.io/issue-13-2/)


### KVO

KVO，即Key-Value-Observe，键值绑定。意思就是观察一个对象的内容变化并响应这个变化。

当变化发生的时候，KVO notification 会立即调用观察者实现的`observeValueForKeyPath:ofObject:change:context:`方法来让观察者及时响应这个通知。

一致地，一个对象要想获取另一个对象的变化通知，必须要先注册成为这个对象的观察者。
通过`addObserver:forKeyPath:options:context:`方法来注册成为观察者。

KVO的可以解决那些修改不是在本模块，但是又想获知修改内容的场景。比如经常见到的列表上拉下拉刷新控件，就需要注册观察UITableView的contentOffset等属性来获取这个变化并及时做调整刷新控件位置及状态。

### target-action

命令（target-action）模式在Cocoa有大量应用。Cocoa中通过target-action设计模式来实现UIControl对象与其他对象之间的沟通。手机用户界面中包含大量可供交互的对象（UIControl），例如UIButton、UISlider等。control对象的职责就是负责解释用户的意图并找到合适的对象去实现这个意图。当用户触摸了设备中的一个按钮的时候，一个触摸事件就会产生，control对象负责接收这个触摸事件，并负责创建相应的操作。事件本身并不知道用户确切的意图，所以需要通过target-action方案，将事件与事件相关的操作联系起来。

通过调用这个方法`addTarget:action:forControlEvents:`来绑定这个关系。

* Target：是消息的接收者，经常可以是controller。
* Action：当消息转发到target的时候，target对这个消息的实现。下面就是一个controller对一个按钮点击事件的实现：

``` 
- (IBAction)buttonClicked:(UIButton *)button;
```

### 工厂方法

工厂方法：由类实现的快捷生成器，是加方法，在这个快捷方法中组合了alloc和init，并返回创建好的对象。

可以参考NSArray中的工厂方法：
``` 
+ (instancetype)array;
+ (instancetype)arrayWithObject:(ObjectType)anObject;
```

我们经常说Objective-C中的对象有两步创建的特点。即要先alloc分配空间，然后再init作初始化操作。而工厂方法其实就是对两步创建的包装，使得调用者可以更快捷的得到一个可用的对象。开发者可以参考Cocoa开发框架中对工厂方法的使用，对调用者提供一些快捷的工厂方法。

### 类簇

类簇是`抽象工厂`模式在iOS下的一种实现，众多常用类，如NSString，NSArray，NSDictionary，NSNumber都运作在这一模式下，它是接口简单性和扩展性的权衡体现，在我们完全不知情的情况下，偷偷隐藏了很多具体的实现类，只暴露出简单的接口。

详细的说明可以参考这篇文章：[从NSArray看类簇](http://blog.sunnyxx.com/2014/12/18/class-cluster/)

### Blocks

#### Blocks的使用

Blocks是c语言的托冲功能，是带有自动变量的匿名函数。

``` 
int foo = 10;
^(int event) {
	printf(“foo : %d”, foo);
	printf(“eventId : %d”, event);
}
```

Blocks的特点：匿名，解决了命名这一难题。可以捕捉上下文环境，很好的解决了异步回调问题。

Blocks的原理过于复杂，这里不作讲解。学习iOS前期可以只看看Blocks的引用对我们的编程方式带来哪些变化，以及使用Blocks过程中需要注意的一些问题。

在接口设计方面，使用Blocks之前，苹果在设计UIAlertView类的时候，通过UIAlertViewDelegate协议声明了 - (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
这个方法，来供使用者响应用户的点击事件。
在iOS8.0后，苹果设计了UIAlertController替代UIAlertView。对于事件的响应方式作出了这样的变化：

```
+ (instancetype)actionWithTitle:(nullable NSString *)title style:(UIAlertActionStyle)style handler:(void (^ __nullable)(UIAlertAction *action))handler;
```
可以看到handler这个参数是个Blocks块，接受的是我们对事件点击的处理函数。

相比以前使用代理的方式，使用Blocks块，可以在创建UIAlertController的同时设置响应点击的异步回调，并且在这个回调Blocks中可以轻松捕捉上下文变量，给编程带来了很大的方便。

#### Blocks的循环引用

如果在Block中使用附有__strong修饰符的对象类型自动变量，那么当Block从栈复制到堆上时，该对象被Block所持有。这样就容易引起循环引用

```
typedef void (^blk_t)(void);

@interface MyObject : NSObject

@property (nonatomic, copy) blk_t block;

@end

@implementation MyObject

- (void)dealloc {
    NSLog(@"myobj dealloc");
}

- (id)init {
    self = [super init];
    if (!self) {
        return nil;
    }
    _block = ^ {
        NSLog(@"self = %@", NSStringFromClass([self class]));
    };

    return self;
}
@end
```

MyObject类对成员变量block指向的Block是强引用关系，而这个Block中又自动截获了__strong修饰的self，会同时再持有self。self持有Block，而Block持有self，造成了循环引用。
解决方法是在Block中使用__strong类型变量时，先用__weak修饰符修饰变量。

```
typeof(self)  __weak tmp = self;
_block = ^ {
	NSLog(@"self = %@", NSStringFromClass([tmp class]));
};
```

## 附件下载及参考资料

* 附件01: [MVC-Demo](https://redharry.b0.upaiyun.com/download/MVCDemo.zip)

* 推荐阅读：[cocoa设计模式官方文档](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html)


## 下次课程概要


iOS中的网络请求











