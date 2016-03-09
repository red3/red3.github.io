---
layout: post
title:  "iOS学习沙龙系列（七）：iOS中网络请求的正确姿势"
date:   2016-3-9 18:30:00
---


客户端应用想要出色，离不开网络的支持。而网络请求又是个比较复杂的过程，因为涉及到多端的协作。在实际的工作中，后台负责提供数据，客户端显示数据，客户端需要向后台发起请求以拿到正确的数据。而测试团队不仅要测试客户端是否正确显示后台数据，有时候还需要抓包工具来查看后台接口提供的接口是否合理。不过，也正是因为这种多方协作的感觉，才显得有趣，不是嘛！


## 同步 or 异步

### 同步请求方案

iOS中同步的请求方案一般由具体返回值的类来封装实现，比如同步请求一个字符串，可以调用NSString的下面的这个方法：
```
NSString *string = [[NSString alloc] initWithContentsOfURL:url encoding:NSUTF8StringEncoding error:nil];
```
请求NSData值，可以调用由NSData实现的下面的这个方法：
```
NSData *data = [NSData dataWithContentsOfURL:url];
```

需要注意的是因为这些初始化方法是同步请求，在主线程中发起同步请求容易产生阻塞，故一般推荐使用异步请求方案。

### 异步请求方案

iOS中一般通过`NSURLRequest`发起异步请求，`NSURLConnection`利用这个请求来创建连接：

```
NSURLRequest *request = [NSURLRequest requestWithURL:url];
NSURLConnection *httpConnection = [[NSURLConnection alloc] initWithRequest:request delegate:self];
```
既然请求是异步的，必然就得有方法使得我们接受到请求的数据。NSURLConnection 通过代理的形式，将请求结果下发到具体的调用者，也就是上述代码中看到的NSURLConnection 初始化方法中的delegate参数。

请求连接一旦创建后，就开始下载数据。下载过程的数据处理，通过下面的代理方法来处理：

```
// 收到服务器的响应的时候，回调这个方法
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    // 可以在收到服务器响应的时候，初始化用于接收数据的容器
    _downloadPipeline = [[NSMutableData alloc] init];
}

// 接收数据过程，多次调用这个方法
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    // 请求的数据都保存在_downloadPipeline中
    [_downloadPipeline appendData:data];
}

// 数据接收完成，回调这个方法
- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
}

// 连接失败，回调这个方法
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
{
    NSLog(@"error = %@",error);
}
```

通过代理回调函数，我们可以在数据请求成功后更新UI界面，或者在请求失败的时候，弹窗提示用户。

## GET or POST

我们知道，HTTP请求有GET 和 POST 之分，那么在iOS中是怎么体现GET 或者 POST的呢？通过上面的文章，我们已经知道请求是通过NSURLRequest发起的。所以甭管GET 还是 POST 肯定逃不过使用NSURLRequest就对了，只是在NSURLRequest的基础上，两者有些差异。

### GET

GET请求是将请求参数拼接在URL里面的，所以跟前文的请求一样，只是需要在请求URL中拼接上请求参数就是了。一般的请求URL大概是这个样子：
> http://127.0.0.1/index.php?realName=xxx&nickName=yyy

**?**后面开始拼接的都是参数对了，参数对之间用**&**符号拼接，参数对中间，**=**符号前面是参数名，后面是对应的值。

所以只需要将请求参数拼接到URL里面，然后正常发起请求就行：

```
NSURL *url = [NSURL URLWithString:@"http://127.0.0.1/index.php?realName=xxx&nickName=yyy"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
NSURLConnection *httpConnection = [[NSURLConnection alloc] initWithRequest:request delegate:self];
```

### POST

区别NSURLRequest，`NSMutableURLRequest`用来创建可变请求，可变请求可以修改请求方式，所以可以将请求方式由默认GET方式修改为POST。下面的示例代码说明如何创建POST请求。

```
// 这个是post请求参数对
NSDictionary *paras = @{@"realName": @"xxx",
                        @"nickName": @"yyy"};
                           
NSURL *url = [NSURL URLWithString:@"xxx"];
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
// 默认请求方式是GET， 需要修改为POST
request.HTTPMethod = @"POST";

// 用=号拼接参数
NSMutableArray *buffer = [[NSMutableArray alloc] init];
for (NSString *key in paras) {
	id obj = [paras objectForKey:key];
	NSString *str = [NSString stringWithFormat:@"%@=%@",key,obj];
	[buffer addObject:str];
}
// 用&号拼接参数对
NSString *combinedString = [buffer componentsJoinedByString:@"&"];
// 转换成二进制数据
NSData *data = [combinedString dataUsingEncoding:NSUTF8StringEncoding];
// 设置请求体
[request setHTTPBody:data];

```
实质是，上述方法就是将请求参数转换成二进制数据附带在请求的请求体里面。有时候需要设置请求的请求头，也是通过类似方法，即`setValue:forHTTPHeaderField:`方法来完成。


## 数据解析（json转model）

拿到的请求数据一般为NSData格式，所以我们首要的任务是先将NSData数据作json解析，解析出json里面的数据对应关系。

`NSJSONSerialization`是系统提供用来解析json数据的类，通过该类可以将json数据解析成NSArray或者NSDictionary （取决于json数据的格式）。

```
// JSON（数据格式）解析
NSData *data;  // 接口请求的json数据
NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:nil];
   
```
json解析后，我们得到键值对应的数据，通常为NSDictionary，通常我们需要该字典格式转换成MVC模式中的Model。通常的作法是将字典中的每个key对应的值取出来赋值给model中对应的属性：

```
@interface UserModel : NSObject
@property (nonatomic, copy) NSString *nickName;
@end

{
 NSDictionary *para = @{@"realName": @"xxx",
                        @"nickName": @"yyy"};
 NSString *nickName = para[@"nickName"];
 UserModel *model = [[UserModel alloc] init];
 model.nickName = nickName;
}
```
可以看出，当model中属性很多的时候，通过这样的方法来赋值是效率很低下的事情。所幸的是，苹果的开发团队已经想到了这个问题，并且也给出了解决方案：

```
@interface UserModel : NSObject
@property (nonatomic, copy) NSString *nickName;
@property (nonatomic, copy) NSString *realName;
@end

{
 NSDictionary *para = @{@"realName": @"xxx",
                        @"nickName": @"yyy"};
 NSString *nickName = para[@"nickName"];
 UserModel *model = [[UserModel alloc] init];
 [model setValuesForKeysWithDictionary:para];
}
```

可以看到，只需要一个方法`setValuesForKeysWithDictionary:`就可以将字典中的键值关系一一对应到model中的属性中去，很强大有木有。而这一切这个强大的基础，就是Cocoa提供了一套`KVC`键值编码的技术，既然是键值编码，有一点就不得不提，就是我们能正确一一赋值的前提是字典的key正好就是model中的属性名称，要不然就没有对应关系，也就没法给属性赋值了。有兴趣的童鞋可以自行了解下KVC这个强大的技术。

顺带提一点，实际开发过程中，因为各个语言之间编码习惯的差异，服务器接口返回的字典中的key值，并不总是能跟model中的属性名称相符。比如上述代码中的realName字段，可能在后台后以`real_name`这个字段命名。所以客户端的开发经常需要处理这些键值有差异的情况。也可以参考一些开源的解析框架，比如：YYModel，JSONModel，Mantle等。



## 业界普遍采用的网络框架方案

目前几乎所有的iOS开发者在选择网络框架的时候，都会选择[AFNetworking](https://github.com/AFNetworking/AFNetworking)来作为其基础框架，截止今天（2016.03.09），其在github上就已经有23500+的star了，足见其深得iOS开发者喜爱。

AFNetworking有充分的开发文档说明应该怎么使用它，摘抄一段接口请求的方法：

```
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
AFURLSessionManager *manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];

NSURL *URL = [NSURL URLWithString:@"http://httpbin.org/get"];
NSURLRequest *request = [NSURLRequest requestWithURL:URL];

NSURLSessionDataTask *dataTask = [manager dataTaskWithRequest:request completionHandler:^(NSURLResponse *response, id responseObject, NSError *error) {
    if (error) {
        NSLog(@"Error: %@", error);
    } else {
        NSLog(@"%@ %@", response, responseObject);
    }
}];
[dataTask resume];
```

需要说明的是，`NSURLSessionDataTask`继承于`NSURLSessionTask`，NSURLSession是自iOS7.0以后苹果框架中提供的网络请求方案,NSURLConnection在iOS9被宣布弃用，并且最新的AFNetworking也放弃基于NSURLConnection的连接方案得继续维护了。
