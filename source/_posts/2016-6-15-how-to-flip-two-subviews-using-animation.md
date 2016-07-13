title: iOS 中如何实现翻转动画特效
tags:
date:   2016-06-15 18:30:00
---

先上一个动图，阐述下本文所谓的翻转动画到底是怎样的效果？

![](http://photo-coder.b0.upaiyun.com/img/how-to-flip-two-subviews-using-animation01.gif)

乍一看很懵逼，翻来覆去的，以前完全没有搞过这种动画啊。

那我们就先复习下，我们可以搞定的动画在 iOS 中一般是用哪些 API：

```
Block 版本
+ (void)animateWithDuration:(NSTimeInterval)duration animations:(void (^)(void))animations completion:(void (^ __nullable)(BOOL finished))completion NS_AVAILABLE_IOS(4_0);

```

或者是这样的：

```
事务模式
[UIView beginAnimations:nil context:nil];
[UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
[UIView setAnimationDuration:1];
// Apply the animation to the backdrop 
[UIView commitAnimations];
```

相比较来说，Block版本的 API 使用起来更加方便，只需要将动画变化放入 Block块中就可以了，其实 Block 版的 API 内部还是用事务实现的。

利用上面的 API ，我们可以完成 iOS 开发中绝大多数情况下的动画效果。

值得为 UIKit 团队喝彩的是，本文的这种效果同样包含在这绝大多数情况里面。仔细阅读 UIView 头文件里面关于动画的部分发现这个api：

```
+ (void)setAnimationTransition:(UIViewAnimationTransition)transition forView:(UIView *)view cache:(BOOL)cache;  
```
意即如果你想在变换两个子 View 的时候加上反转动画效果，可以使用这个 API. 并且苹果在接口注释里面还给出了一个示例告诉我们如何做出这样的动画效果：

```
[fromView removeFromSuperview]; 
[containerView addSubview:toView];
```

事实上这个接口也是有block版本的，对应的 API 是：

```
+ (void)transitionWithView:(UIView *)view duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options animations:(void (^ __nullable)(void))animations completion:(void (^ __nullable)(BOOL finished))completion NS_AVAILABLE_IOS(4_0);

```
并且在 iOS4.0 后苹果更加推荐用这种方式来做类似的动画效果。

完整的动画效果代码应该是这样的：

```
UIView *toView;
UIView *fromView;
UIViewAnimationOptions option =  UIViewAnimationOptionTransitionFlipFromLeft;
void (^update)(void) = ^ {
       // reload data
    };
[UIView transitionWithView:self.containerView
                      duration:0.5f
                       options:option 
                    animations:^ {
                        [fromView removeFromSuperview]; 
						[containerView addSubview:toView];
                    }
                    completion:^ (BOOL finished){
                        update();
                    }];
```

目前来看，这个动画效果基本实现了。但是还有点问题，拿我们这个例子来说，假如用户当前在 TableView 上浏览商品，为了良好的用户体验，当用户点击切换视图按钮的时候，我们想把用户在 TableView 上的浏览轨迹(浏览到第几个条目)同步到 CollectionView 上，该怎么做？

我想到的办法是在 ScrollView 的滑动减速的代理里面同步当前浏览位置，但是依据目前的实现，当我们切换浏览视图的视图，会把上一个视图从父视图上移除，所以即使我拿到了当前浏览视图的浏览位置，也无法同步到另一个视图上（因为这个视图并不在屏幕上）。

顺着这个思路，解决办法就是把 Tableview 和 CollectionView 都加到父视图上，在初始化的时候调用这个 API 改变两个视图的 Z 轴位置：

```
[containerView insertSubview:collectionView belowSubview:tableView];

```

对应的，在做动画的 Block 块里，切换两个视图的 Z 轴位置：

```
[containerView exchangeSubviewAtIndex:0 withSubviewAtIndex:1];

```
至此，我们完美解决了问题！

BTW，我们还可以使用 Core Animation 框架 `CATransition` 类来做类似的动画效果，具体该怎么实现，就留读者自己探索吧。





