
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

对上面的知识点了解以后，我们需要把定时器跟波浪公式以及UIScrollView结合起来创建出一条动态的波浪曲线。

饭要一口一口吃，代码要一步一步写，我们先创建出一条正弦曲线出来：

```objective-c
UIView *view = [[UIView alloc] initWithFrame:CGRectMake(0, 100, self.view.frame.size.width, 200)];
view.backgroundColor = [UIColor whiteColor];
[self.view addSubview:view];
CAShapeLayer *firstWaveLayer = [CAShapeLayer layer];
firstWaveLayer.fillColor = [UIColor lightGrayColor].CGColor;
CGMutablePathRef path = CGPathCreateMutable();
CGFloat y = 50;
CGPathMoveToPoint(path, nil, 0, y);
CGFloat waveWidth = self.view.frame.size.width;
CGFloat cycle = 6 * M_PI / self.view.frame.size.width;
CGFloat offsetX = 0;
for (float x = 0.0f; x <=  waveWidth ; x++) {
    y = 8 * sin(cycle * x + 0) + 70 ;
    CGPathAddLineToPoint(path, nil, x, y);
}
CGPathAddLineToPoint(path, nil, waveWidth, 100);
CGPathAddLineToPoint(path, nil, 0, 100);
CGPathCloseSubpath(path);
firstWaveLayer.path = path;
CGPathRelease(path);
[view.layer addSublayer:firstWaveLayer];
```

当然仅仅只有一条正弦曲线是模拟不出来波浪的效果的，还需要一条余弦曲线才可以合成波浪曲线效果：

```objective-c
CAShapeLayer *secondWaveLayer = [CAShapeLayer layer];
secondWaveLayer.fillColor = [UIColor redColor].CGColor;
CGMutablePathRef path = CGPathCreateMutable();
CGFloat y = 50;
CGPathMoveToPoint(path, nil, 0, y);
for (float x = 0.0f; x <=  waveWidth ; x++) {
    y = 8 * cos(cycle * x + offsetX) + 70 ;
    CGPathAddLineToPoint(path, nil, x, y);
}
CGPathAddLineToPoint(path, nil, waveWidth, 100);
CGPathAddLineToPoint(path, nil, 0, 100);
CGPathCloseSubpath(path);
secondWaveLayer.path = path;
CGPathRelease(path);
[view.layer addSublayer:secondWaveLayer];
```

然后我们可以看见效果是这样的：


![](http://photo-coder.b0.upaiyun.com/img/waveview-with-cadisplaylink07.png)

从图中可以看出，相同参数下的正弦曲线和余弦曲线并不能很好的合成一个对称的曲线，我们想要的效果是正弦曲线的波峰对应余弦曲线的波谷，所以需要将余弦函数的水平便宜做一个调整。


![](http://photo-coder.b0.upaiyun.com/img/waveview-with-cadisplaylink03.jpeg)

从图中可以看出，标准的余弦函数需要在水平方向上向左偏移四分之一周期的距离才能够跟同参数的正弦函数对称。

```objective-c
CGFloat offsetX = M_PI/cycle/2;  // also equal 2*M_PI/_cycle/4;
```

![](http://photo-coder.b0.upaiyun.com/img/waveview-with-cadisplaylink04.png)

现在波浪有了，要想让波浪动起来，需要有定时器每次触发的时候都产生两条新的曲线（path），然后替换现有曲线，就跟老式的放电影的思路一样，快速替换达到动态的效果。

先创建定时器，然后给定时器绑定上事件：

```objective-c
displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(displayLinkTric)];
[displayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
```

为了形成动态效果，我们需要每次产生曲线的时候都有一个水平方向的偏移量，让产生的曲线每次都比上次偏移一点：

```objective-c
- (void)displayLinkTric {
    
    static CGFloat offsetX = 0;
    offsetX += 0.07;
    
    CGFloat waveWidth = self.view.frame.size.width;
    CGFloat cycle = 6 * M_PI / self.view.frame.size.width;
    
    {
        CGMutablePathRef path = CGPathCreateMutable();
        CGFloat y = 50;
        CGPathMoveToPoint(path, nil, 0, y);
        
        
        for (float x = 0.0f; x <=  waveWidth ; x++) {
            y = 8 * sin(cycle * x + offsetX) + 70 ;
            CGPathAddLineToPoint(path, nil, x, y);
        }
        CGPathAddLineToPoint(path, nil, waveWidth, 100);
        CGPathAddLineToPoint(path, nil, 0, 100);
        CGPathCloseSubpath(path);
        firstWaveLayer.path = path;
        CGPathRelease(path);
    }
    
    {
        CGMutablePathRef path = CGPathCreateMutable();
        CGFloat y = 50;
        CGFloat forword = M_PI/cycle/2;  // also equal 2*M_PI/_cycle/4
        CGPathMoveToPoint(path, nil, 0, y);
        for (float x = 0.0f; x <=  waveWidth ; x++) {
            y = 8 * cos(cycle * x + offsetX + forword) + 70 ;
            CGPathAddLineToPoint(path, nil, x, y);
        }
        CGPathAddLineToPoint(path, nil, waveWidth, 100);
        CGPathAddLineToPoint(path, nil, 0, 100);
        CGPathCloseSubpath(path);
        secondWaveLayer.path = path;
        CGPathRelease(path);
    }
}

```

最终可以看到流动的波浪产生了：

![](http://photo-coder.b0.upaiyun.com/img/waveview-with-cadisplaylink05.gif)

仔细观察，发现波浪还是不够逼真，因为真实的播放不仅是前进的，还是浮动的，所以我们的这个波浪缺少了浮动的感觉，前面在正弦函数的部分提起过，要改变正弦函数的波动，需要改变它的振幅，所以需要一个算法来动态产生一个振幅：

```objective-c

- (void)displayLinkTric {
    
    static CGFloat offsetX = 0;
    offsetX += 0.05;
    
    static CGFloat amplitude = 8;
    static BOOL increase = YES;
    
    if (increase) {
        amplitude += 0.04;
    } else {
        amplitude -= 0.04;
    }
    
    if (amplitude >= 12) {
        increase = NO;
    }
    if (amplitude <= 4) {
        increase = YES;
    }
    
    CGFloat waveWidth = self.view.frame.size.width;
    CGFloat cycle = 2 * M_PI / self.view.frame.size.width;
    
    {
        CGMutablePathRef path = CGPathCreateMutable();
        CGFloat y = 50;
        CGPathMoveToPoint(path, nil, 0, y);
        
        
        for (float x = 0.0f; x <=  waveWidth ; x++) {
            y = amplitude * sin(cycle * x + offsetX) + 70 ;
            CGPathAddLineToPoint(path, nil, x, y);
        }
        CGPathAddLineToPoint(path, nil, waveWidth, 100);
        CGPathAddLineToPoint(path, nil, 0, 100);
        CGPathCloseSubpath(path);
        firstWaveLayer.path = path;
        CGPathRelease(path);
    }
    
    {
        CGMutablePathRef path = CGPathCreateMutable();
        CGFloat y = 50;
        CGFloat forword = M_PI/cycle/2;  // also equal 2*M_PI/_cycle/4
        CGPathMoveToPoint(path, nil, 0, y);
        for (float x = 0.0f; x <=  waveWidth ; x++) {
            y = amplitude * cos(cycle * x + offsetX - forword) + 70 ;
            CGPathAddLineToPoint(path, nil, x, y);
        }
        CGPathAddLineToPoint(path, nil, waveWidth, 100);
        CGPathAddLineToPoint(path, nil, 0, 100);
        CGPathCloseSubpath(path);
        secondWaveLayer.path = path;
        CGPathRelease(path);
    }
}
```

主要的思想就是通过一个布尔值控制振幅的增长，当增长到了最高值的时候让振幅减小，减小到最低值的时候再增长，以此来产生一个动态的振幅，然后就会看到下面的效果了：

![](http://photo-coder.b0.upaiyun.com/img/waveview-with-cadisplaylink06.gif)


至此，其实核心的开发已经完成了，剩下的就是通过UIScrollView的偏移量来计算出一个动态的波浪振幅：

```objective
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    
        CGFloat offset = (-scrollView.contentOffset.y-scrollView.contentInset.top);
        CGFloat times = offset/10 + 1;
}
```

可以用计算出的`times`变量来动态控制振幅的变化。

通过UIScrollView来动态控制振幅的难点在与不能通过UIScrollView的代理来实现具体的算法，因为不能把View层的东西冗余到Controller层去，秉承良好的设计模式，需要给UIScrollView实现一个拓展方法，在拓展方法里面让我们实现波浪函数的View添加为UIScrollView的观察者，在观察到UIScrollView的offset每次变化时，动态计算振幅，具体的实现还是在源码中了解吧。

最后，完整的项目地址在这里：[HHPullToRefreshWave](https://github.com/red3/HHPullToRefreshWave)







