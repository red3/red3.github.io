---
layout: post
title:  "iOS学习沙龙系列（二）：新建一个工程以及Foundation、UIKit"
date:   2016-1-20 13:05:00
---





## 前言

### 补充

* OS X使用指南

    1. OS X两个窗口并排显示的支持：点击并按住一个窗口的全屏按钮，随后软件建议将相关应用显示在另一侧

* 学习iOS经典书籍补充

    1. [Zen and the Art of the Objective-C Craftsmanship](https://www.gitbook.com/book/yourtion/objc-zen-book/details) : Objective-C style 代码该怎么写，代码指南，如何设计和构建优秀的代码

    2.  [禅与Objective-C编程艺术](https://www.gitbook.com/book/yourtion/objc-zen-book-cn/details) : 上面这本书的中文翻译版本，Objective-C style 代码该怎么写，代码指南，如何设计和构建优秀的代码

* iOS社区博客推荐（这个会持续更新）

    * [OneV's Den](http://onevcat.com/#blog) ： 王巍 (@onevcat)，一名来自中国的 iOS / Unity 开发者。现居日本，就职于 LINE。正在修行，探求创意之源，并且有在维护一个Swift使用技巧的网站

    * [唐巧的技术博客](http://devtang.com/) ： InfoQ编辑, 《iOS开发进阶》作者, 在猿题库创业

    * [sunnyxx](http://blog.sunnyxx.com/)  ：90后非主流iOS程序猿，现负责百度知道iOS团队。视代码为颜值，待节操为路人，追寻最佳实践，爱刨根问底



* iOS社区公众号推荐（这个会持续更新）

    * InfoQ ：有内容的技术社区媒体，微信号：infoqchina

    * iOS开发 ： 唐巧维护，微信号：iosDevTips 


## 本期纲要

### 开启iOS编程之旅 

#### 新建工程

* `shitf`+`command`+`N` 新建工程 ->
* 选择模板：iOS app 或者 command line tools
* 填写工程信息：bundle id是在AppStore中唯一标示一个应用的手段，一般以这样的形式 com.xxx.yourProjectName，其中`xxx`为公司组织的英文名。
* 如果选择了iOS app，接下来会在工程目录中看到·`main.storyboard`文件，称之为故事板，你对编程的所有想象从这里展开。

#### 查看文档

在初始阶段，你对iOS的认知必定是苍白的，但是，但凡你了解到你将要做的事情的一丁点信息，就可以在文档中检索这些信息，来获得全面、权威的帮组。
关于如何查看文档，你需要知道的：

* `shift`+`command`+`0`: 查看完整的文档引用
* `command`+`单击类名或者属性名`: 进入改类的接口说明文件
* `option`+`单击类名或者属性名`: 弹出改类或者属性的说明

#### 提升工作效率的快捷键

以下这些快捷键是我根据我自己的日常工作需要，按照使用频率整理出来的一份列表，以供参考：

##### 编码中

* `command`+`R`：运行

* `command`+`单击类名或者属性名`：查看文档

* `option`+`单击类名或者属性名`：查看文档

* `command`+`N`：新建文件

* `command`+`F`：当前文件中查找

* `command`+`3`：当前项目中查找

* `shift`+`command`+`O`：快速进入某一文件或者类接口文档等

* `command`+`/`：增加或者取消注释

* `command`+`L`：跳转行号

* `control`+`command`+`上方向键`：接口文件与实现文件切换


* `control`+`I`：重新整理缩进

* `control`+`1`：显示相关条目，查看callers 和 callees tree

* `control`+`6`：显示文档下条目，快速查看以及跳转

##### XIB中

* `control`+`单击控件` 显示该控件距离其他控件的距离

* `option`+`command`+`0`：显示或者隐藏utilities 功能区，修改控件属性，或者添加控件等。

* `option`+`command`+`回车`：切换至assistant editor界面，方便将控件的引用拖到代码中

### Objective-C 与 Swift、Storyboard与手写代码

Xcode目前支持的语言有C++, C, Objective-C, Swift。

关于是否选择Swift作为入门语言，我的建议是不，理由是：

1. Swift尚不稳定，几乎每次Xcode更新说明中都会提及对Swift的bug的修复
2. ABI（应用二进制接口）
3. 苹果原生应用中几乎没有用Swift写的应用

但是并不意味着不可以学习Swift，相反，我们应该主动着手去学习这门语言，因为他是一门优秀的语言，是未来的趋势。

关于选择StoryBoard还是手写代码

我的建议是选择Storyboard和XIB，从目前苹果对Xcode和iOS的更新来看，每次更新都会带来XIB新的特性支持，
并且XIB是苹果推荐的方式，是开发方向，前期可以完全在XIB中布局控件，但是后期一定要学会怎么用代码创建UI


### Foundation框架介绍

可以通过附件01 project来简单了解Objective-C`Foundation `框架：

``` objective-c
        NSObject *object = [[NSObject alloc] init];
        
        // String
        // NSString
        {
            // constant
            NSString *string0 = @"i";
            NSString *string1 = @"OS";
            NSString *string2  = [string0 stringByAppendingString:string1];
            BOOL isEqual = [string0 isEqualToString:string2];
            
        }
        // NSMutableString
        {
            // variable
            NSMutableString *mutableString = [NSMutableString stringWithString:@"iOS"];
            // add
            [mutableString appendString:@"and Android"];
            // delete
            [mutableString deleteCharactersInRange:[mutableString rangeOfString:@"and Android"]];
            // update
            [mutableString replaceCharactersInRange:[mutableString rangeOfString:@"iOS"]
                                         withString:@"Android"];
            // select
            NSString *string = [mutableString substringWithRange:NSMakeRange(0, 2)];
            NSLog(@"%@", string);
        }
        
        // Data
        // NSData
        {
            // constant
            NSString *string = @"testString";
            NSData *data = [string dataUsingEncoding:NSUTF8StringEncoding];
        }
        
        // NSMutableData
        {
            // variable
            NSString *string = @"testString";
            NSData *data = [string dataUsingEncoding:NSUTF8StringEncoding];
            NSMutableData *mutableData = [NSMutableData dataWithData:data];
            // add
            [mutableData appendData:data];
            // delete
            [mutableData resetBytesInRange:NSMakeRange(0, 2)];
            // update
            [mutableData replaceBytesInRange:NSMakeRange(0, 5) withBytes:[string cStringUsingEncoding:NSUTF8StringEncoding]];
            // select
            NSData *selectedData = [mutableData subdataWithRange:NSMakeRange(0, 5)];
            NSLog(@"%@", selectedData);
        }
        
        // Dic
        // NSDictionary
        {
            // constant
            NSDictionary *dict = @{@"code": @"200",
                                   @"msg": @"success"};
            // select
            NSString *msg = dict[@"msg"];
            NSString *code = [dict objectForKey:@"code"];
            
        }
        {
            // variable
            NSDictionary *dict = @{@"code": @"200",
                                   @"msg": @"success"};
            NSMutableDictionary *mutableDict = [[NSMutableDictionary alloc] initWithDictionary:dict];
            // add && update
            [mutableDict setObject:@"content" forKey:@"result"];
            // delete
            [mutableDict removeObjectForKey:@"result"];
        }
        
        
        // Array
        // NSArray
        {
            // constant
            NSArray *array = @[@"obj0", @"obj1", @"obj2"];
            // select
            NSString *str0 = [array objectAtIndex:0];
            NSUInteger index = [array indexOfObject:str0];
            // traver
            for (NSString *string in array) {
                NSLog(@"%@", string);
            }
        }
        {
            // variable
            NSMutableArray *mutableArray = [[NSMutableArray alloc] initWithCapacity:2];
            // add
            [mutableArray addObject:@"obj0"];
            [mutableArray addObject:@"obj1"];
            [mutableArray addObject:@"obj2"];

            // delete
            [mutableArray removeObject:@"obj0"];
            [mutableArray removeObjectAtIndex:0];
            // update
            [mutableArray replaceObjectAtIndex:0 withObject:@"obj3"];
        }
        
        // Date
        // NSDate
        NSDate *date = [NSDate date];
        NSLog(@"current date : %@", date);
        
        // NSFileManager
        NSFileManager *fileManager = [NSFileManager defaultManager];
        NSData *fileData = [@"data" dataUsingEncoding:NSUTF8StringEncoding];
        NSString *path = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject] stringByAppendingPathComponent:@"file.txt"];
        [fileManager createFileAtPath:path contents:fileData attributes:nil];
        [fileManager fileExistsAtPath:path];
        // NSError
        
        // NSNull
        NSNull *null = [NSNull null];
        // NSBundle
        [[NSBundle mainBundle] pathForResource:@"img" ofType:@"png"];
```


### UIKit介绍

附件02 project是对UIKit中的以下基本控件做的简单介绍，可以通过这个工程对iOS中的控件做一个基本的了解。

* UIButton: 提供特定的交互动作，按钮，可设置图片，文字

* UILabel：显示静态文本，可显示多行文本

* UIActivityIndicatorView： 指示任务的进度（常用于图片加载过程，上传文件过程）

* UIDatePicker：显示日期和时间，比如小时，分钟，天，月，年等，日历中有应用

* UIPageControl：分页控制器，指示当前页数，iOS设备主屏下部应用，拉手团购首页广告banner

* UIPickerView：可供选择值的集合，修好了项目选择故障方案有应用

* UIProgressView：进度指示，具有明确的进度时间，iOS邮件程序有应用

* UISegmentedControl：提供类似button功能的按钮的线性集合，用于切换不同的显示内容，iOS设备AppStore程序有应用

* UISwitch：开关控件，提供两种选择，设置界面

* UITextField: 接受用户单行输入，各个应用的登录界面， 在备课过程中发现keyboardmode

* UIAlertView: 提供App使用操作的重要说明信息，toast，浮层，位于界面中间

* UIActionSheetView：提供App使用操作的重要说明信息，toast，浮层，位于界面底部



## 附件下载

* [附件01](https://redharry.b0.upaiyun.com/download/Foundation_01.zip): Foundation框架介绍
* [附件01](https://redharry.b0.upaiyun.com/download/UIKitDemo.zip): UIKit框架介绍




## 下次课程概要

contentView，bar，Viewcontroller













