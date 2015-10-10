
---
layout: post
title:  "关于3DTouch开发者需要了解的事情"
date:   2015-10-06 14:30:00
---

2015年的秋季新品发布会上苹果在新一代的iPhone6S上增加了名为3DTouch新一代交互方式。通过3DTouch，屏幕可以感知手指触摸在屏幕上的力度，用于解决手机平台长期缺乏类似PC上"右键"功能。

而对于开发者来说，苹果公开了3个新的接口，允许开发者在自己的应用程序中加入这个革命性的交互方式。
苹果在Apple Watch上加入Force Touch，一如其名，即设备可以感知用户拇指触压的力度。3D Touch对应在pc平台的操作有点类似于右键。苹果增加了3个api允许开发者在自己的应用中使用这个新的交互，`Quick Actions`允许在应用程序icon上按压以提供最多为4个的快捷方式；`Peek`允许在不脱离当前页面内容的情况下提供某一页面的预览视图，`Pop`则发生在当用户对某一Peek感兴趣时，继续按压，以进入该预览视图。

本文旨在通过一个Demo程序来说明开发者要响应这个新的交互特性，需要做哪些事情。




## Home Screen Quick Actions

`Quick Actions`使得用户可以快速跳转到应用程序的某个地方而省去以往繁琐的步骤。这个地方，对于短信程序来说，可以是一个"新建信息"的快捷方式，对于电话程序来说，可以是一个"拨打某一号码"的快捷方式，可以想象，这些类似的特性是确切地方便了用户的特性，真的可以让用户放声笑，笑出声。

需要了解的是`Quick Actions`分为静态类型与动态类型。在该Demo程序中，我添加了4个快捷方式，分别为一个静态的shortcutTitle1，3个动态的new、search、share。

![](http://photo-coder.b0.upaiyun.com/img/img_3D_touch1.png)

静态类型只需要在Info.plist文件中添加一条key为"UIApplicationShortcutItems"的记录即可。该key对应的记录的格式如下：
```
<array>
	<dict>
		<key>UIApplicationShortcutItemIconType</key>
		<string>UIApplicationShortcutIconTypeSearch</string>
		<key>UIApplicationShortcutItemSubtitle</key>
		<string>shortcutSubtitle1</string>
		<key>UIApplicationShortcutItemTitle</key>
		<string>shortcutTitle1</string>
		<key>UIApplicationShortcutItemType</key>
		<string>First</string>
		<key>UIApplicationShortcutItemUserInfo</key>
		<dict>
			<key>firstShorcutKey1</key>
			<string>firstShortcutKeyValue1</string>
		</dict>
	</dict>
</array>

```
而动态类型需要在Appdelegate中添加，意即`application`有一个`shortcutItems`的属性，允许开发者动态该属性中的内容。代码如下：

``` Objective-C
if (application.shortcutItems == 0) {
        UIMutableApplicationShortcutItem *shortcut2 = [[UIMutableApplicationShortcutItem alloc] initWithType:@"Second" localizedTitle:@"new" localizedSubtitle:nil icon:[UIApplicationShortcutIcon iconWithType:UIApplicationShortcutIconTypeAdd] userInfo:nil];

        UIMutableApplicationShortcutItem *shortcut3 = [[UIMutableApplicationShortcutItem alloc] initWithType:@"Third" localizedTitle:@"search" localizedSubtitle:nil icon:[UIApplicationShortcutIcon iconWithType:UIApplicationShortcutIconTypeSearch] userInfo:nil];
        UIMutableApplicationShortcutItem *shortcut4 = [[UIMutableApplicationShortcutItem alloc] initWithType:@"Fourth" localizedTitle:@"share" localizedSubtitle:nil icon:[UIApplicationShortcutIcon iconWithTemplateImageName:@"icon_share.png"] userInfo:nil];

        
        application.shortcutItems = @[shortcut2, shortcut3, shortcut4];
    }
```
同以往处理远程通知类似，我们通过`-application:performActionForShortcutItem:completionHandler:`代理方法来处理从快捷方式启动的事件，对于应用程序的首次启动即是通过快捷方式启动的特殊情况，我们需要在`-application:didFinishLaunchingWithOptions:`方法中返回`NO`来避免应用程序去调用上面的那个方法。



## Peek and Pop

类似于Mac平台上空格键的预览功能，在手机上可以预览类似图片、视频以及网址等。通过轻按某一条项目，peek可以提供某一条项目的预览以及想关联的一些操作，而不用离开当前页面，手指松开，peek view自动消失。当用户通过预览界面发现是自己感兴趣的内容，可以在peek view上稍重一点的按压，会进入详情页面。

关于这两个接口需要注意的点：

- 通过peek提供实时且内容充足的预览。因为需要给用户提供足够的信息来让其判断是否要继续进入这个页面
- 为每一个peek提供一个pop
- 当实现了peek的时候就不要提供 Edit menu，因为两种操作方式很相近，容易迷惑用户
- 不要把peek作为唯一的交互方式。因为老设备不支持3DTouch

在最新的开发者文档中可以看到这两个接口有一个名为`UIViewControllerPreviewingDelegate`代理对象。对于具体的编码工作来说，要将一个需要支持3DTouch的页面视图(UIView)注册成为该接口的代理，当用户触发了3D Touch，该代理会响应到该触摸的位置，即一个处在刚才注册的UIView上的`CGPoint`，开发者依据改位置，决定是否有一个peek应该呈现。

``` Objective-C
- (nullable UIViewController *)previewingContext:(id <UIViewControllerPreviewing>)previewingContext viewControllerForLocation:(CGPoint)location
{
    // Obtain the index path and the cell that was pressed.
    // guard let indexPath = tableView.indexPathForRowAtPoint(location),
    NSIndexPath *indexPath = [self.tableView indexPathForRowAtPoint:location];
    UITableViewCell *cell = [self.tableView cellForRowAtIndexPath:indexPath];
    if (!cell) {
        // no peek
        return nil;
    }
    
    // Create a detail view controller and set its properties.
    DetailViewController *detail = [[DetailViewController alloc] init];
    NSDictionary *previewDetail = self.sampleData[indexPath.row];
    detail.previewingTitle = previewDetail[@"title"];
    
    /*
     Set the height of the preview by setting the preferred content size of the detail view controller.
     Width should be zero, because it's not used in portrait.
     */
    detail.preferredContentSize = CGSizeMake(0, [previewDetail[@"preferredHeight"] doubleValue]);
    // Set the source rect to the cell frame, so surrounding elements are blurred.
    previewingContext.sourceRect = cell.frame;
    return detail;
}
```
具体来说，需要处理这几件事：

- 将CGPoint转换成UITableView中的`NSIndexPath`
- 依据NSIndexPath找到对应的`Cell`
- 创建`DetailViewController`并复制相关属性
- 处理DetailViewController的`preferredContentSize`属性，该属性的width参数应该设置为0，因为在竖屏模式下该参数并不启用

当用户决定`Pop`该预览视图的时候：

``` Objective-C
- (void)previewingContext:(id <UIViewControllerPreviewing>)previewingContext commitViewController:(UIViewController *)viewControllerToCommit
{
    // Reuse the "Peek" view controller for presentation.
    [self showViewController:viewControllerToCommit sender:self];
}
```

## Preivew Controllers

苹果在其开发文档中指出，在上文提到的delegate中返回的UIViewcontroller不具备交互能力，意味着即使你看见这个视图中有button等元素，也不能响应button的点击，也不能在这个视图上添加手势等操作。作为替代方案，苹果提供了`UIPreviewActionItem`，类似`UIAlertController`中的Acton的实现。

![](http://photo-coder.b0.upaiyun.com/img/img_3D_touch2.gif)

要实现这已功能，覆盖`-previewActionItems:`方法，并返回一个包含`UIPreviewAction`的数组即可：

``` Objective-C
- (NSArray <id <UIPreviewActionItem>> *)previewActionItems
{
    // set the previewItems
    // avoid the retain cycel
    // __weak MainViewController *weakSelf = self;
    UIPreviewAction *item1 = [UIPreviewAction actionWithTitle:@"Share" style:UIPreviewActionStyleDefault handler:^(UIPreviewAction * _Nonnull action, UIViewController * _Nonnull previewViewController) {
        
    }];
    UIPreviewAction *item2 = [UIPreviewAction actionWithTitle:@"Delete" style:UIPreviewActionStyleDefault handler:^(UIPreviewAction * _Nonnull action, UIViewController * _Nonnull previewViewController) {
        
    }];
    return @[item1, item2];
}
```

## 结尾

希望通过本文可以帮助到正在集成3DTouch特性的开发者。

最后放上github的repo：[3DTouchDemo](https://github.com/red3/3DTouchDemo)



