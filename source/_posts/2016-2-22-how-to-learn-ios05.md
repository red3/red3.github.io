---
layout: post
title:  "iOS学习沙龙系列（五）：UITableView计算高度的那些事以及内存管理"
date:   2016-2-22 20:28:00
---

## UITableView计算高度的那些事

### 怎么计算高度

UITableView 是个 UIScrollView，就像平时使用 UIScrollView 一样，加载时需要指定 contentSize 后它才能根据自己的 bounds、contentInset、contentOffset 等属性共同决定是否可以滑动以及滚动条的长度。而 UITableView 在一开始并不知道自己会被填充多少内容，于是询问 data source 个数和创建 cell，同时询问 delegate 这些 cell 应该显示的高度，这就造成它在加载的时候浪费了多余的计算在**屏幕外边**的 cell 上。所以该小节内容主要围绕UITableViewCell的高度计算以及计算高度机制所带来的性能问题来讨论下业界的一些解决方案。

* 定高的解决方案-rowHeight

```objective-c
// will return the default value if unset
@property (nonatomic) CGFloat rowHeight;             
```

指定cell高度有两种方式：rowHeight以及代理方法`-tableView: heightForRowAtIndexPath:`方法

定高cell需求推荐使用rowHeight属性，代理方法计算高度使用在动态高度的cell中。


* 不定高，手动计算高度的方案：

通过NSString的分类方法，来计算多行文本的高度

```objective-c
@interface NSString (NSExtendedStringDrawing)
- (CGRect)boundingRectWithSize:(CGSize)size options:(NSStringDrawingOptions)options attributes:(nullable NSDictionary<NSString *, id> *)attributes context:(nullable NSStringDrawingContext *)context NS_AVAILABLE(10_11, 7_0);
@end

```

使用起来是这个样子的：

```objective-c

NSDictionary *attribute = @{NSFontAttributeName: xxx};
size = [text boundingRectWithSize: xxx
			   options: NSStringDrawingTruncatesLastVisibleLine | 					     NSStringDrawingUsesLineFragmentOrigin | 					     NSStringDrawingUsesFontLeading
             attributes:attribute
             context:nil].size;
```

使用这个API，需要预先向字符串指定要显示的UILabel的宽度以及字号等信息，从设计模式的角度理解，感觉接口设计的比较丑陋。

* 不定高，iOS6以后Auto layout的解决方案

从iOS6开始，可以使用Autolayout来计算cell高度：

	- (CGSize)systemLayoutSizeFittingSize:(CGSize)targetSize;
                                 
通过cell的实例自己去计算当前cell的约束去适应cell中文本、图片等信息后应该呈现的高度。缺点是计算速度肯定没有手算快，而且这是个实例方法，需要维护专门为计算高度而生的 template layout cell，它还要求使用者对约束设置的比较熟练，要保证 contentView 内部上下左右所有方向都有约束支撑，设置不合理的话计算的高度就成了0。
                         
* 不定高，iOS8时代 self-sizing cell

旨在让 cell 自己负责自己的高度计算，使用 frame layout 和 auto layout 都可以享受到。但是只在ios8之后才可用。

### 计算高度带来的性能问题

* 估算高度-estimatedHeight

段首提到，UITableView在加载的时候浪费了多余的计算在屏幕外边的 cell 上，在存在大量cell的时候，或者cell布局复杂的时候，这个计算时间要耗费大量时间。所以iOS7以后应运而生这个属性，旨在让UITableView加载的时候通过这个估算高度来计算自身的contentSize.

contentSize.height 根据“cell估算值 x cell个数”计算

但是这个解决方案也存在问题：

- 设置估算高度后，contentSize.height 根据“cell估算值 x cell个数”计算，这就导致滚动条的大小处于不稳定的状态，contentSize 会随着滚动从估算高度慢慢替换成真实高度，肉眼可见滚动条突然变化甚至“跳跃”。

- 若是有设计不好的下拉刷新或上拉加载控件，或是 KVO 了 contentSize 或 contentOffset 属性，有可能使表格滑动时跳动。

- 估算高度设计初衷是好的，让加载速度更快，那凭啥要去侵害滑动的流畅性呢，用户可能对进入页面时多零点几秒加载时间感觉不大，但是滑动时实时计算高度带来的卡顿是明显能体验到的，个人觉得还不如一开始都算好了呢（iOS8更过分，即使都算好了也会边划边计算）

* iOS8时代开始的坑爹的算高机制

iOS7时代，当cell的高度计算出来以后，继续滑动，是不会重复计算cell的高度的，而在iOS8时代，继续滑动仍然会重复调用算高函数，这样做的原因是cell被认为可以随时变化高度的，比如在设置里面调节了系统的字号，所以每次滑动都要重新计算cell高度。因此，iOS8时代，算高函数大量调用，极其影响性能。

### 解决方案
鉴于上述，我们想有一个省去算高麻烦，又能保证顺畅的滑动，还能支持 iOS6+ 的解决方案：

[UITableView+FDTemplateLayoutCell](https://github.com/forkingdog/UITableView-FDTemplateLayoutCell)

百度知道团队开源的这个解决方案通过UITableview的拓展，从设计模式层面上，ViewController为了计算高度而产生的模板cell被迁移到了UITableview层， 通过autolayout 约束自动计算高度，并且计算出的cell的高度也由UITableview缓存了起来。


## 内存管理

Objective-C通过自动引用计数的方式来管理内存，自动引用计数（ARC，Automatic Reference Counting）是指内存管理中对引用采取自动计数的计数。

> 在LLVM编译器中设置ARC为有效状态，就无需再次键入retain或者是release代码了。

需要指出的是，在Xcode4.2或之后才可以使用自动引用计数的技术，而在之前需要程序员需要手动管理内存。

而自动内存管理也是可以关闭的：

1.针对整个工程build setting 搜`gar` 将**Objective-C Automatic Reference Counting** 这个属性值 改为YES
2.针对单一文件build phases-compile sources中定位相应的文件设置其compiler flag为 `-fno-objc-arc` (相反，要指定该文件生效-fobjc-arc)

### 引用计数

Objective-C的引用计数，可以用开关房间的灯为例来说明这个机制。

上班进入办公室的人需要照明，所以把灯打开，下班离开的人，不需要照明了，所以把等关掉。多人上班的时候需要使办公室在至少还有1人的情况下保持开灯，无人时保持关灯，判断是否有人在办公室，通过计数功能来计算"需要照明的人数"。也就是说：

1. 第一个人进入办公室，"需要照明的人数"加1，计数值从0变为1，要开灯
2. 之后每次有人进入，"需要照明的人数"就加1。如计数值从1变为2
3. 每当有人离开办公室，"需要照明的人数"就减1，如计数值从2变为1
4. 最后的人离开，"需要照明的人数"减1，计数值从1变为0，因此关灯。

在Objective-C中，对象相当于办公室的照明设备，通过类似这种管理照明设备的方式，管理内存。

#### 内存管理的思考方式

关于引用计数这个名词，我们应该采用的客观的、正确的思考方式是：

- 自己生产的对象，自己持有
- 非自己生成的对象，自己也能持有
- 不再需要自己持有的对象时释放
- 非自己持有的对象无法释放

##### 自己生成的对象，自己持有

使用一下名称开头的方法名意味着自己(自己是指对象的使用环境，也可以理解为程序员自身)生成的对象只有自己持有：
 
 * alloc
 * new
 * copy：通过copyWithZone方法生成并持有对象的副本
 * mutableCopy：类比copy方法，生成的是可变更的对象

``` objective-c
// 自己创建的对象，自己持有
id obj = [[NSObject alloc] init];

```
 
##### 非自己生成的对象，自己也能持有

采用上述方法alloc/new/copy/mutableCopy之外取得的对象，因为非自己生成并持有，所以自己不是该对象的持有者，需要通过`retain`方法，获得对象的持有权。

``` objective-c
// 对象存在，自己不持有
id obj = [NSMutableArray array];
// 需要持有对象的话，retain它
[obj retain];

```

##### 不再持有对象时释放对象

自己持有的对象，一旦不需要的时候，持有者有义务释放该对象。释放对象使用release方法。对象一经释放不可再次访问。

``` objective-c
// 自己创建的对象，自己持有
id obj = [[NSObject alloc] init];
// 不再需要对象的时候，释放对象
[obj release];

```

##### 无法释放非自己持有的对象

释放非自己持有的对象会造成程序崩溃

``` objective-c
// 自己创建的对象，自己持有
id obj = [[NSObject alloc] init];
// 不再需要对象的时候，释放对象
[obj release];
// 自己不再持有对象，再次释放对象会造成崩溃
[obj release];

```

#### 通过实现dealloc来释放持有的对象

NSObject 类 提供了dealloc方法，这个方法会在对象的引用计数为0的时候自动调用（这个方法不能主动调用），可以在这个方法里面释放该类持有的一些对象。

```objective-c
@interface Person : NSObject
@property (retain) NSString *firstName;
@property (retain) NSString *lastName;
@property (assign, readonly) NSString *fullName;
@end
 
@implementation Person
- (void)dealloc
    [_firstName release];
    [_lastName release];
    // 注意最后必须调用父类的dealloc方法
    [super dealloc];
}
@end
```

ARC中，编译器会自动实现类的dealloc方法，在这个方法里面释放强引用的变量引用，程序员不能显式调用dealloc方法，但是可以实现dealloc方法以作取消注册通知等操作。

#### autorelease以及NSAutoreleasePool的一点说明

前面提到[NSMutableArray array]这个方法返回的对象自己不持有，那么这个方法内部是怎么创建的对象？

``` objective-c
- (NSMutableArray *)array {
	// 自己创建并持有对象
	NSMutableArray *array = [[NSMutableArray alloc] init];
	// 在函数返回之前，需要把自己创建的对象释放
	// 但是问题是释放以后，调用者就无法使用这个对象了
	// 所以这种情况下需要使用autorelease来管理内存
	[array autorelease];
	return array;
}

```

从上例可以看到，用autorelease，可以使得对象存在，但自己不持有对象。autorelease提供这样的功能，使对象在超出指定的生存范围时能够自动并正确地释放，也就是延迟释放的功能。

这个正确的释放时机，通过自动释放池（autoreleasepool）来完成，注册在这个池子里的对象，会在自动释放池结束的时候去释放池子里的资源。

在ARC中，也是通过autoreleasepool来管理内存的，编译器会在函数的作用域中**自动插入**自动释放池代码块，位于这个代码块中的对象，在释放池末尾的时候，会自动对对象发送autorelease

```objective-c
@autoreleasepool {
        // ...
    }
```


## ARC规则
引用计数式的内存管理的思考方式同样适用思考ARC所引起的变化。

- 自己生产的对象，自己持有
- 非自己生成的对象，自己也能持有
- 不再需要自己持有的对象时释放
- 非自己持有的对象无法释放

这种方式在ARC有效时也是可行的，只是在编写代码的时候记述方法会有所不同。


### 所有权修饰符

* __strong
* __weak

#### __strong修饰符

__strong修饰符是id类型和对象的默认修饰符

``` objective-c
id obj = [[NSObject alloc] init];
// 这行代码相当于
id __strong obj = [[NSObject alloc] init];

```

__strong修饰符表示"强引用"关系。持有强引用的变量在超出其作用域的时候被废弃，废弃的方式是通过Xcode自动在其作用域结束的时候加上release代码。

#### __weak修饰符

__weak修饰符设计出来解决一些循环引用的问题。

* 对象A强引用了对象B
* 对象B也强引用了对象A

```objective-c
@interface TestWeekObj : NSObject
{
    id __strong _obj;
}

- (void)setObject:(id __strong)obj;

@end

@implementation TestWeekObj

- (void)setObject:(id __strong)obj {
    _obj = obj;
}

- (void)test {
    // test0持有对象A的强引用
    TestWeekObj *test0 = [[TestWeekObj alloc] init]; // 对象A
    // test1持有对象B的强引用
    TestWeekObj *test1 = [[TestWeekObj alloc] init]; // 对象B
    
    [test0 setObject:test1];
    [test1 setObject:test0];
    // 截止现在
    // 对象A的强引用变量为test0 以及对象B的_obj
    // 对象B的强引用变量为test1 以及对象A的_obj
    //
    /*
     * 超出作用域，强引用失效，释放变量
     * 变量test0 超出作用域，释放对象A
     * 变量test1 超出作用域，释放对象B
     * 但是对象B的_obj变量还强引用对象A，故对象A无法释放
     * 但是对象A的_obj变量还强引用对象B，故对象B无法释放
     */
}

```

__weak修饰与__strong修饰符相反，提供弱引用。弱引用不持有对象实例。
上面的例子中，只要把成员变量_obj的修饰符换成__weak，即可打破循环引用。

### 代码规则

* 不可使用retain/release/autorelease
* 遵守内存管理的方法命名规则
* 不可显示调用dealloc

属性中的修饰规则，下面是一些示例

```objective-c
// 在ARC中
@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) NSArray *array;
@property (nonatomic, weak) id delegate;
@property (nonatomic) CGFloat height;

```

```objective-c
// ARC禁用的时候
@property (nonatomic, copy) NSString *name;
@property (nonatomic, retain) NSArray *array;
@property (nonatomic, assign) id delegate;
@property (nonatomic) CGFloat height;

```



## 附件下载及参考资料

* 附件01: [内存管理Demo](https://redharry.b0.upaiyun.com/download/AboutMemoryMgmt.zip)

* 推荐书籍：[iOS与OS X多线程和内存管理](http://item.jd.com/11258970.html)
* 推荐阅读：[内存管理官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmRules.html)


## 下次课程概要


iOS中常用的设计模式（MVC，代理，单例）以及使用Block块进行对象间的通讯











