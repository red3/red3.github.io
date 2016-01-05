
---
layout: post
title:  "通过CADisplayLink开发带波浪效果的下拉刷新"
date:   2015-12-10 15:02
---

阿里的蚂蚁聚宝app中发现了很酷炫的下拉刷新效果，大概是这个样子的：

![](http://photo-coder.b0.upaiyun.com/img/alipay-waveview-lookslike.gif)

然后自己动手做了这个效果，所以这篇文章是为了讲解[HHPullToRefreshWave](https://github.com/red3/HHPullToRefreshWave)这个项目是如何产生的。

## 需要预先了解的概念（Understanding）

### CADisplayLink


```objective-c
+ (CADisplayLink *)displayLinkWithTarget:(id)target selector:(SEL)sel;
- (void)addToRunLoop:(NSRunLoop *)runloop forMode:(NSString *)mode
```
是定时器，每次屏幕刷新的时候会调用SEL执行特定的方法。跟`NSTimer`一样，可以加入到runloop中。

```objective-c
@property(readonly, nonatomic) CFTimeInterval duration;
```

duration只读，每帧之间的时间，每次刷新之间的时间间隔。可以通过这个时间计算下一帧要显示的的UI的数值。需要注意的是 duration 只是大概的时间。如果cpu忙于其他计算，会跳过几次回调。

```objective-c
@property(nonatomic) NSInteger frameInterval;
```
frameInterval可读写，隔多少帧调用一次selector，默认1，意即每帧调用一次，iOS刷新频率60HZ，意即每秒调用60次。


### 正弦波浪公式

> y=Asin(ωx+φ)+k

* A——振幅，当物体作轨迹符合正弦曲线的直线往复运动时，其值为行程的1/2
* ω——角速度， 控制正弦周期(单位角度内震动的次数)
* φ——初相，x=0时的相位；反映在坐标系上则为图像的左右移动
* k——偏距，反映在坐标系上则为图像的上移或下移

根据公式：改变A，可以让波浪幅度改变，ω目前来看应该是个定值，因为他控制的是角速度，波浪曲线的角速度只在特定的一些值下才显得自然。改变φ使得曲线有水平位移效果

### CAShapeLayer

```objective-c
@property(nullable) CGPathRef path;
```
可以依据正弦波浪曲线的函数，创建出一条CGMutablePathRef实例，然后通过CAShapeLayer显示出这条曲线。

## 联合使用

对上面的知识点了解以后，我们需要把定时器跟波浪公式以及UIScrollView结合起来创建出一条动态的曲线：






