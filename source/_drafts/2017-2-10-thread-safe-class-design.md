---
layout: post
title:  "设计线程安全的类"
date:   2017-2-1 18:30:00
---


https://www.objc.io/issues/2-concurrency/thread-safe-class-design/

### 线程安全

#### 苹果的 Frameworks

首先，让我们来看一下苹果提供的 Framework. 一般来讲，大多数的类都是默认非线程安全的，除非做特别的声明。有时候这种默认的做法是 xxxxxx

即使有经验的 iOS/Mac 开发者也会犯的一个错误是在其他线程访问 UIKit/AppKit 的资源，这个错误常常发生在在后台线程设置诸如 `image` 等属性，因为这行属性的内容是通过网络请求在后台下载的。苹果对代码做了性能优化，因此当你在不同的线程中改变属性的时候，它并不会报警。

就拿这个图片的例子来说，常见的症状(在后台线程设置 image 属性后)是设置生效要延迟一些，但是，如果两个线程同时设置了图片，很有可能你的程序会直接崩溃，因为当前被设置过的图片会被释放两次。由于这种情况取决于时间，很有可能在你开发的时候不会崩溃但是到用户手中的时候就会崩溃。

官方并没有提供一个工具来检测这样的错误，但是有一些技巧可以来胜任这个工作。[UIKit Main Thread Guard]()里面提供的代码可以勾住 UIView 的 `setNeedsLayout` `setNeedsDisplay` 并且在转发之前检查是否是执行在主线程上的。由于 UIKit 的绝大多数属性修改（包括 image）都会导致这两个方法被调用，因此，这个方法可以捕捉到很大部分的线程相关的错误。尽管这个技巧并没有用到私有的 API, 我们并不推荐在生成环境中也使用它 --这开发环境中就足够了。

对于 UIKit 不是线程安全这个事情，苹果这边是有意为之的。把它设计成线程安全的并不会带来更多好处，相反却很消耗性能；实际上，这会拖慢很多事情。事实上，把 UIKit 绑定到主线程上使得并发编程和使用 UIKit 都非常容易，你只需要确保对 UIKit 的调用都发生在主线程上即可。

#### 为什么 UIKit 不能是线程安全的？

确保庞大的动态库诸如 UIKit 是线程安全的是一项浩大的工程并且会带来很大的开销，把 nonatomic 变为 atomic 仅仅是这里面的一小部分。通常你想要同时改变数个属性，然后才看到结果，针对这种情况，苹果将不得不增加类似 CoreData 的 `performBlock:` 和 `performBlockAndWait:` 方法来同步改变。并且，如果你认为多数对 UIKit 的调用是为了配置属性，那么线程安全则变得不那么有意义。

然而，即使一些调用不是为了更改共享的内部状态也是非线程安全的。如果你在 iOS3.2 和之前的坑爹时代写过 apps 的话，你应该体验过调用 NSString 的 `drawInRect:withFont:` 带来的崩溃问题 xxxxxx，谢天谢地，从 iOS4 开始，[苹果将大部分和绘制有关的方法和类像 UIColor 和 UIFont 设计成可以使用在后台线程](https://developer.apple.com/library/content/releasenotes/General/WhatsNewIniOS/Introduction/Introduction.html)

不幸的是，苹果的文档缺少关于线程安全的主题，苹果仅仅是推荐所有的访问应该发生在主线程，甚至对于绘制相关的方法，他们也不保证线程安全，所以，经常读 [Release Notes](https://developer.apple.com/library/content/releasenotes/General/WhatsNewIniOS/Articles/iPhoneOS4.html)不失为好的习惯。

总结来说，UIKit 中的类只应该在主线程中使用，对与一些从 UIResponder 派生出来的类和一些涉及操作应用程序 UI 的类，这条准则尤其适用。

#### Deallocation 问题

在后台线程使用 UIKit 会带来另一个危险事情：Deallocation 问题。苹果这这个 [issue](https://developer.apple.com/library/content/technotes/tn2109/_index.html) 中指出了这个问题，并给出了一些解决方法。这个问题的关键是 UI 对象必须要在主线程释放，因为在 dealloc 中，它们可能带来视图层级的变化，正如我们之前说的，这些变化是需要发生在主线程的。

由于在次要的线程，block 块中持有 UI 资源是很普遍的做法，就很容易发生这样的错误，且不容易发现和修改，这个也是 [AFNetworking 长期以来的一个bug](https://github.com/AFNetworking/AFNetworking/issues/56), 仅仅由于很少有人了解到这个 issue 并且通常这个 issue 也会带来偶然的，难以复现的崩溃。常见的解决方案是使用 __weak 修饰符并且不要在异步的 block 或 线程中访问 ivars.

#### Collection Classes

关于常用的 foundation 类的线程安全列表，苹果提供了一份[概述](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/ThreadSafetySummary/ThreadSafetySummary.html#//apple_ref/doc/)以供查询。一般来将，不可变类型例如 NSArray 是线程安全的，而它们对应的可变类型例如 NSMutableArray 是非线程安全的。实际上，在不同线程中使用它们是没有问题的，只要保证读写操作是序列化在同一个队列中即可。需要谨记的是方法的返回值可能是可变类型及时声明的时候是不可变类型，好的实践是通过类似 `return [array copy]` 代码来确保返回的是不可变类型。

不像 Java 等编程语言，Foundation 框架并不提供开箱即用的线程安全的集合类。这样做是可以理解的，因为大多数情况下














