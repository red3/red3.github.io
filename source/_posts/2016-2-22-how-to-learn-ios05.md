---
layout: post
title:  "iOS学习沙龙系列（五）：UITableView计算高度的那些事以及内存管理"
date:   2016-2-22 20:28:00
---

## UITableView计算高度的那些事

### 怎么计算高度

UITableView 是个 UIScrollView，就像平时使用 UIScrollView 一样，加载时需要指定 contentSize 后它才能根据自己的 bounds、contentInset、contentOffset 等属性共同决定是否可以滑动以及滚动条的长度。而 UITableView 在一开始并不知道自己会被填充多少内容，于是询问 data source 个数和创建 cell，同时询问 delegate 这些 cell 应该显示的高度，这就造成它在加载的时候浪费了多余的计算在**屏幕外边**的 cell 上。所以该小节内容主要围绕UITableViewCell的高度计算以及计算高度机制所带来的性能问题来讨论下业界的一些解决方案。

* 定高的解决方案-rowHeight

、、、objective-c
rowHeight
```

cell高度有两种方式：rowHeight以及代理方法

定高需求使用该属性，代理方法计算高度使用在动态高度的cell中

* 不定高，手动计算高度的方案：

NSDictionary *attribute = @{NSFontAttributeName: font};

size = [text boundingRectWithSize:rectSize
                                  options:NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading
                               attributes:attribute
                                  context:nil].size;

* 不定高，iOS6以后Auto layout的解决方案                                  
 
 -systemLayoutSizeFittingSize:
 
 缺点是计算速度肯定没有手算快，而且这是个实例方法，需要维护专门为计算高度而生的 template layout cell，它还要求使用者对约束设置的比较熟练，要保证 contentView 内部上下左右所有方向都有约束支撑，设置不合理的话计算的高度就成了0
                         
* 不定高，iOS8时代 self-sizing cell

旨在让 cell 自己负责自己的高度计算，使用 frame layout 和 auto layout 都可以享受到
确定是只在ios8之后才可用

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


鉴于上述，我们想有一个省去算高麻烦，又能保证顺畅的滑动，还能支持 iOS6+ 的解决方案

UITableView+FDTemplateLayoutCell






## 下次课程概要

内存管理













