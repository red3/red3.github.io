---
layout: post
title:  "8种方案帮助你摆脱笨重的ViewController"
date:   2016-3-19 20:30:00
---


I am Soroush Khanlou, and this is my blog. I write about programming, primarily Objective-C and learning what I can from other languages, like Ruby and Haskell. You can catch me talking about photography, politics, eating, and fonts.

Get in touch via Twitter @khanlou or email soroush@khanlou.com.



View controllers 变得臃肿因为他们要处理太多的事情。管理键盘，用户输入，数据转换，创建view - 这些事情真的是view controller的负责范围吗？这里面的哪些是可以委托个其他的对象的？在这篇文章，我们将探索依据这些事情各自的职责来分离到他们应该属于的对象里面去。这样做会帮助我们隔离大量**复杂**的代码，并且也使我们的代码更具有可读性。

在view controller里面，我们会依据职责用`#pragma mark`来把代码划分成不同的组。通常，当我们这样做的时候，就是一个很好的时机来思考将这些代码迁移到更小的组件中去。

## Data Source

**Data Source 设计模式**是隔离业务逻辑与？？？？应用在复杂的table ivews的时候，可以用来从view controller移除诸如"在当前坐标下哪些cell是可见的"等逻辑问题。如果你需要经常写一些判断table view的行或组的代码，设计一个数据源对象是合适的。

这个数据源对象可以只遵守 `UITableViewDataSource` 协议，因为我发现配置cell对应的model对象跟管理cell的index path是不同的分工，所以我喜欢把这两个职责分离开来。

一个简单的例子，数据源对象的可以处理分组逻辑：

```
@implementation SKSectionedDataSource : NSObject

- (instancetype)initWithObjects:(NSArray *)objects sectioningKey:(NSString *)sectioningKey {
    self = [super init];
    if (!self) return nil;

    [self sectionObjects:objects withKey:sectioningKey];

    return self;
}

- (void)sectionObjects:(NSArray *)objects withKey:(NSString *)sectioningKey {
    self.sectionedObjects = //section the objects array
}

- (NSUInteger)numberOfSections {
    return self.sectionedObjects.count;
}

- (NSUInteger)numberOfObjectsInSection:(NSUInteger)section {
    return [self.sectionedObjects[section] count];
}

- (id)objectAtIndexPath:(NSIndexPath *)indexPath {
    return self.sectionedObjects[indexPath.section][indexPath.row];
}

@end
```

尽管这个数据源对象应该要设计成抽象的并且可以复用的，也别怕去构建一个只使用在一个地方的数据源对象。将index path逻辑的管理从view controller中分离出去是我们的目标，尤其是那些不断变化的列表，让某一个对象通知控制器"Hey，在这个index path下有新的对象"是可行的，接下来控制器可以传递这个对象到列表并以动画呈现出来。

这一设计模式同样可以应用在拉取数据逻辑上，数据源可以拉取一组对象从远程接口。`UIViewController`是UI对象，所以这么做是很好的方式将网络请求的代码从控制器从分离了出去。

如果你的所有的数据源对象的接口是固定的（比如说通过protocol来声明），那么你可以构建一个特殊的数据源对象？？？

## Standard Composition

通过iOS5以后新增的组合控制器接口可以将View controllers组合起来使用。如果你的视图控制器是由几个组分合成的，其中的每个组分都可以是一个单独的视图控制器，就考虑使用组合模式来将他们分离开来。屏幕上有多个table view或者collection views就是最佳实践。

当屏幕上有一个头部视图和一个网格视图，我们可以使用懒加载这两个视图控制器，然后当系统要绘制他们的时候再正确布局他们：

```
- (SKHeaderViewController *)headerViewController {
    if (!_headerViewController) {
        SKHeaderViewController *headerViewController = [[SKHeaderViewController alloc] init];

        [self addChildViewController:headerViewController];
        [headerViewController didMoveToParentViewController:self];

        [self.view addSubview:headerViewController.view];

        self.headerViewController = headerViewController;
    }
    return _headerViewController;
}

- (SKGridViewController *)gridViewController {
    if (!_gridViewController) {
        SKGridViewController *gridViewController = [[SKGridViewController alloc] init];

        [self addChildViewController:gridViewController];
        [gridViewController didMoveToParentViewController:self];

        [self.view addSubview:gridViewController.view];

        self.gridViewController = gridViewController;
    }
    return _gridViewController;
}

- (void)viewDidLayoutSubviews {
    [super viewDidLayoutSubviews];

    CGRect workingRect = self.view.bounds;

    CGRect headerRect = CGRectZero, gridRect = CGRectZero;
    CGRectDivide(workingRect, &headerRect, &gridRect, 44, CGRectMinYEdge);

    self.headerViewController.view.frame = tagHeaderRect;
    self.gridViewController.view.frame = hotSongsGridRect;
}

```

这样使得每个子视图控制器中的collection视图可以处理一种类型的数据，也因此更容易维护和更改。

## Smarter Views

如果你在视图控制器里面构建你的所有的子视图的话，你可能需要考虑`Smarter View`.

`UIViewController`默认使用`UIView`来作为自己的view属性，但是你可以用你自己的view来重写这个属性。你可以通过`-loadView` 来达到目的，只要你在这个方法里面给`self.view`赋值。

```
@implementation SKProfileViewController

- (void)loadView {
    self.view = [SKProfileView new];
}

//...

@end

@implementation SKProfileView : NSObject

- (UILabel *)nameLabel {
    if (!_nameLabel) {
        UILabel *nameLabel = [UILabel new];
        //configure font, color, etc
        [self addSubview:nameLabel];
        self.nameLabel = nameLabel;
    }
    return _nameLabel;
}

- (UIImageView *)avatarImageView {
    if (!_avatarImageView) {
        UIImageView * avatarImageView = [UIImageView new];
        [self addSubview:avatarImageView];
        self.avatarImageView = avatarImageView;
    }
    return _avatarImageView
}

- (void)layoutSubviews {
    //perform layout
}

@end
```

你只需要重新声明view属性`@property (nonatomic) SKProfileView *view`，并且因为这个新的属性的比UIView更明确，编译器会作正确的识别并辨别出`self.view`是`SKProfileView`类型的。这种[改变返回值类型](https://en.wikipedia.org/wiki/Covariant_return_type)的方法在这种设计模式中非常有用。（编译器需要知道你的这个类是继承于UIView，因此确保导入这个类的头文件而不是仅仅用`@class`声明类。在Xcode6.3以后，你还需要声明这个属性是dynamic的，即在实现区域加入这行代码`@dynamic view;`）

## Presenter

`Presenter`对象接受一个模型对象，然后依据视图上的显示转换它的属性，再将转换过的这些属性暴漏出去（供视图控制器调用）。这种方式同样被用在[Presentation Model](http://martinfowler.com/eaaDev/PresentationModel.html)，[the Exhibit pattern](http://objectsonrails.com/#sec-12_1)，[ViewModel](http://msdn.microsoft.com/en-us/magazine/dd419663.aspx)。

```
@implementation SKUserPresenter : NSObject

- (instancetype)initWithUser:(SKUser *)user {
    self = [super init];
    if (!self) return nil;
    _user = user;
    return self;
}

- (NSString *)name {
    return self.user.name;
}

- (NSString *)followerCountString {
    if (self.user.followerCount == 0) {
        return @"";
    }
    return [NSString stringWithFormat:@"%@ followers", [NSNumberFormatter localizedStringFromNumber:@(_user.followerCount) numberStyle:NSNumberFormatterDecimalStyle]];
}

- (NSString *)followersString {
    NSMutableString *followersString = [@"Followed by " mutableCopy];
    [followersString appendString:[self.class.arrayFormatter stringFromArray:[self.user.topFollowers valueForKey:@"name"]];
    return followersString;
}

+ (TTTArrayFormatter*) arrayFormatter {
    static TTTArrayFormatter *_arrayFormatter;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _arrayFormatter = [[TTTArrayFormatter alloc] init];
        _arrayFormatter.usesAbbreviatedConjunction = YES;
    });
    return _arrayFormatter;
}

@end

```

需要注意的是，模型对象本身并不会暴漏出去（译者注：模型是依赖注入的），Presenter好比模型对象的看门人。这样可以确保视图控制器在不借助persenter对象的情况下不能直接访问到模型对象。类似这样的结构模型会帮你限制依赖关系，因为`SKUser`对象只向很少的类暴漏，修改它会将影响降到最低。

## Binding patter

在方法列表中，这种设计模式？？？？？。当模型数据发生变化时，绑定者负责更新视图。在Cocoa中可以很自然的使用这种方式，因为可以使用KVO来观察模型，使用KVC来获取模型数据并给视图赋值。[Cocoa Bindings](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CocoaBindings/CocoaBindings.html) 是AppKit中对该设计模式的实现。第三方类库例如[Reactive Cocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)事实上也是利用这种技术，只不过更为复杂些。

这种方式结合上面提到的Presenter模式一起使用效果更佳，通过一个类来转换数据，另一个类来把这些转换过的数据传递给视图。

```
@implementation SKProfileBinding : NSObject

- (instancetype)initWithView:(SKProfileView *)view presenter:(SKUserPresenter *)presenter {
    self = [super init];
    if (!self) return nil;
    _view = view;
    _presenter = presenter;
    return self;
}

- (NSDictionary *)bindings {
    return @{
              @"name": @"nameLabel.text",
              @"followerCountString": @"followerCountLabel.text",
            };
}

- (void)updateView {
    [self.bindings enumerateKeysAndObjectsUsingBlock:^(id presenterKeyPath, id viewKeyPath, BOOL *stop) {
        id newValue = [self.presenter valueForKeyPath:presenterKeyPath];
        [self.view setObject:newvalue forKeyPath:viewKeyPath];
    }];
}

@end

```
(需要注意的是上面例子中的Presenter类并不支持KVO，但是是可以做的支持KVO的)

当你第一次尝试使用这种模式的时候，没有一开始致力于设计出完美的抽象模式。不要害怕这个类只会在特定的情形下工作，我们的目标不是说不考虑代码的可复用性，而是让代码更简单，因此这些我们的类就更容易维护，更容易理解。

## Interaction pattern

类似这样的代码`actionSheet.delegate = self`也是让视图控制器变得臃肿的原因。细分来讲，视图控制器的全部职责是接受用户的输入并更新view和model。如今，软件中的交互变得更加复杂，并且这些交互导致视图控制器中出现了臃肿的代码。

当交互包含一个最初的输入（比如轻触按钮），然后弹窗询问用户一个可选的输入（"你确定要X吗？"），然后才产生一个活动，例如网络请求或者状态变更。这个交互的全部过程可以封装到一个*Interaction*类中。下面的示例在按钮点击的时候创建了一个交互类来处理交互，但是通过类似这种`[button addTarget:self.followUserInteraction action:@selector(follow)]`把这个交互类作为按钮动作的target也是可行的。

```
@implementation SKProfileViewController

- (void)followButtonTapped:(id)sender {
    self.followUserInteraction = [[SKFollowUserInteraction alloc] initWithUserToFollow:self.user delegate:self];
    [self.followUserInteraction follow];
}

- (void)interactionCompleted:(SKFollowUserInteraction *)interaction {
    [self.binding updateView];
}

//...

@end

@implementation SKFollowUserInteraction : NSObject <UIAlertViewDelegate>

- (instancetype)initWithUserToFollow:user delegate:(id<InteractionDelegate>)delegate {
    self = [super init];
    if !(self) return nil;
    _user = user;
    _delegate = delegate;
    return self;
}
- (void)follow {
    [[[UIAlertView alloc] initWithTitle:nil
                                message:@"Are you sure you want to follow this user?"
                               delegate:self
                      cancelButtonTitle:@"Cancel"
                      otherButtonTitles:@"Follow", nil] show];
}

- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex {
    if  ([alertView buttonTitleAtIndex:buttonIndex] isEqual:@"Follow"]) {
        [self.user.APIGateway followWithCompletionBlock:^{
            [self.delegate interactionCompleted:self];
        }];
    }
}

@end
```
iOS8以前通过代理工作的alertView 和 actionSheetView 使用这种设计模式会比较有利，当然，该设计模式也同样支持iOS8以后的UIAlertController的接口。
## Keyboard Manager

在键盘状态变更后更新视图是视图控制器中典型的是另一个？？？？？？？，但是这些逻辑可以轻易的迁移到`键盘管理者`中。[关于开源的实现]()被设计出来用于解决这些问题，但是，如果你觉得它过度设计了，不要害怕去实现你自己的键盘管理类。

```
@implementation SKNewPostKeyboardManager : NSObject

- (instancetype)initWithTableView:(UITableView *)tableView {
    self = [super init];
    if (!self) return nil;
    _tableView = tableView;
    return self;
}

- (void)beginObservingKeyboard {
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardDidHide:) name:UIKeyboardDidHideNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillShow:) name:UIKeyboardWillShowNotification object:nil];
}

- (void)endObservingKeyboard {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardDidHideNotification object:nil];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIKeyboardWillShowNotification object:nil];
}

- (void)keyboardWillShow:(NSNotification *)note {
    CGRect keyboardRect = [[note.userInfo objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue];

    UIEdgeInsets contentInsets = UIEdgeInsetsMake(self.tableView.contentInset.top, 0.0f, CGRectGetHeight(keyboardRect), 0.0f);
    self.tableView.contentInset = contentInsets;
    self.tableView.scrollIndicatorInsets = contentInsets;
}

- (void)keyboardDidHide:(NSNotification *)note {
    UIEdgeInsets contentInset = UIEdgeInsetsMake(self.tableView.contentInset.top, 0.0f, self.oldBottomContentInset, 0.0f);
    self.tableView.contentInset = contentInset;
    self.tableView.scrollIndicatorInsets = contentInset;
}

@end
```		

你可以在`-viewDidAppear` 和 `-viewWillDisappear`中分别调用`-beginObservingKeyboard` 和 `-endObservingKeyboard` 或者其他可以的地方。

## Navigator

平时我们通过调用`-pushViewController:animated:`方法来实现不同场景间的导航切换。当页面之间的切换开始变得复杂的时候，可以将这些工作代理给*Navigator*对象。当你在开发iPhone/iPad混合应用时，这种方法尤其有用，导航方式需要根据屏幕尺寸的变化来切换：

```
@protocol SKUserNavigator <NSObject>

- (void)navigateToFollowersForUser:(SKUser *)user;

@end

@implementation SKiPhoneUserNavigator : NSObject<SKUserNavigator>

- (instancetype)initWithNavigationController:(UINavigationController *)navigationController {
    self = [super init];
    if (!self) return nil;
    _navigationController = navigationController;
    return self;
}

- (void)navigateToFollowersForUser:(SKUser *)user {
    SKFollowerListViewController *followerList = [[SKFollowerListViewController alloc] initWithUser:user];
    [self.navigationController pushViewController:followerList animated:YES];
}

@end
@implementation SKiPadUserNavigator : NSObject<SKUserNavigator>

- (instancetype)initWithUserViewController:(SKUserViewController *)userViewController {
    self = [super init];
    if (!self) return nil;
    _userViewController = userViewController;
    return self;
}

- (void)navigateToFollowersForUser:(SKUser *)user {
    SKFollowerListViewController *followerList = [[SKFollowerListViewController alloc] initWithUser:user];
    self.userViewController.supplementalViewController = followerList;
}
@end
```

这样做凸显了使用多个小部件而不是使用一个大部件的好处，它们可以很快地被修改，重写，或者移除。相比较在视图控制器中写那些丑陋的条件判断代码，你只需要在iPad模式下给`self.navigator`赋值为`[SKiPadUserNavigator new]`就可以了，这个navigator会响应同样的协议方法`-navigateToFollowersForUser:`. [告诉他，而不是问他](https://pragprog.com/articles/tell-dont-ask)

## Wrapping up

从历史上看，苹果的SDK只包含组件的最低限度，并且这些API导致你写出臃肿的视图控制器。通过细分你的视图控制器的职责，将可以抽象的组分分离出来，来创造出真正的单责任的对象，我们就可以开始管理这些单个的组件，使得视图控制器再次便于维护。


