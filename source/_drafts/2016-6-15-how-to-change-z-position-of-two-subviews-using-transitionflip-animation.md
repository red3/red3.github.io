title: 如何在 iOS 中实现翻转动画特效
tags:
date:   2016-06-15 18:30:00
---

先上一个动图，阐述下本文所谓的翻转动画到底是怎样的效果？

![](ht)

乍一看很懵逼，翻来翻去的，以前完全没有搞过啊。

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
并且这种方式也是苹果推荐的。
