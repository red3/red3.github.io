title: iOS 中如何实现翻转动画特效
tags:
date:   2016-06-15 18:30:00
---

先上一个动图，阐述下本文所谓的翻转动画到底是怎样的效果？

![](http://photo-coder.b0.upaiyun.com/img/how-to-flip-two-subviews-using-animation01.gif)

乍一看很懵逼，翻来覆去的，以前完全没有搞过这种动画啊。

那我们就先复习下，我们可以搞定的动画在 iOS 中一般是怎样实现的：

```
block 版本
```

或者是这样的：

```
事务模式
```

block版本的api 感觉比较方便，其实 uikit 内部还是用 事务实现的，只不过为了开发者好用，提供了block版本。

通过上面的 API 不难发现，当我们要做动画时，只需要把我们的动画变化（在这之前我们需要了解view的哪些属性是可以做动画的）包裹在动画事务的 begin 和 end 之间就可以了，通过这种方式，我们可以完成 iOS 开发中绝大多数情况下的动画效果。

值得为uikit团队喝彩的是，本文的这种效果同样包含在这绝大多数情况里面。仔细阅读 UIView 头文件里面关于动画的部分发现这个api：

```
+ (void)setAnimationTransition:(UIViewAnimationTransition)transition forView:(UIView *)view cache:(BOOL)cache;  
```


意即If you want to change the appearance of a view during a transition—for example, flip from one view to another—then use a container view, an instance of UIView, as follows:

事实上trans版本也是有block版本的，对应的api是

```
+ (void)transitionWithView:(UIView *)view duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options animations:(void (^ __nullable)(void))animations completion:(void (^ __nullable)(BOOL finished))completion NS_AVAILABLE_IOS(4_0);

```
并且在 iOS4.0 后苹果更加推荐用这种方式来做类似的动画效果。

目前来看，这个动画效果基本实现了。那么，有没有其他方式来实现同样的效果？

当然是有的，先看看代码：

```
CATransition *animation = [CATransition animation];
animation.delegate = self;
animation.duration = 3.0f;
animation.timingFunction = UIViewAnimationCurveEaseInOut;
animation.type = @"oglFlip";
animation.subtype = kCATransitionFromLeft;
// Perform some change of the view 




[self.view.layer addAnimation:animation forKey:@"flip"];
    
```

在上面的代码中，我们用到了 `CATransition` 类，是 Core Animation 框架中用于做视图转换的子类，苹果为该类提供了预先定义的多种转换行为，出现在开发文档中的仅仅只是是 Fade, MoveIn, Push, Reveal, 其余的动画效果可以从[这个链接](http://iphonedevwiki.net/index.php/CATransition)中了解。  

我们的动画效果属于 `oglFlip `, 所以我们直接用这个字符串来赋值 CATransition 实例的 type 属性。

