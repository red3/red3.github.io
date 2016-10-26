---
layout: post
title:  "从 SDWebImage 入手看如何做图片性能优化"
date:   2016-10-5 18:30:00
---

## 前言

注：文中出现的 SD 皆指代 SDWebImage, SDWebImage 是 iOS 开发中有名的异步下载图片框架，其官网地址：[SDWebImage](https://github.com/rs/SDWebImage)

想必做过一段时间 iOS 的同学应该都熟悉 SD，应该也都在自己的项目中使用它，甚至可以说是如何与网络打交道的项目都不会少了它，那么到底这个开源库为什么这么流行？今天我们就从图片性能优化的角度去了解了解这个项目！

在开始之前尼，我想先让大家回想一下，自己在面试的时候有没有被面试官问到过一些关于 SD 的问题，譬如原理啦，实现啦一些的，还记得你是怎么回答的吗，你可能会说它有一个单例 Mangaer 在统筹管理，首先会从内存中或磁盘上查找对应的缓存，如果缓存命中就返回，没有缓存才继续下载，这是 manager 会调用自己持有的 `SDWebImageDownloader` 对象来下载图片，这个 downloader 会相应存储每个下载对应的 progresssBlock 和 CompletionBlock，实际下载是通过实例化一个 NSMutableURLRequest 和 SDWebImageDownloaderOperation, 并将后者加入 downloader 持有的下载队列来完成图片的异步下载。当图片下载完成后，会在主线程设置图片或者返回图片，然后还会将图片缓存在内存中和磁盘中。

相信我，你能这样回答说明你已经很不错了，但是如果这个时候这个面试官好像还是不太满足你说的这些，很有可能他想问的就是今天我们要讨论的关于 SD 里面图片性能优化的部分了，下面我们就带着这个问题，正式进入主题。

## 关于图片性能优化的一些理论知识

图片性能优化只是 iOS 性能优化中的一部分，详细的关于性能优化的知识大家可以移步[这个博客]()，强烈推荐这个博客，我从里面学到很多东西。
这个博客中提到关于图片性能优化的部分有涉及到 cpu 的图片解码：

> iOS 中正常创建的 UIImage 并不会立即解码，当图片设置到 UIImageView 中去的时候，并且 CALayer 被提交到 GPU 前才会进行解码。这个解码过程发生在主线程，不可避免。


那我们就要从这个地方着手来优化图片解码的性能。还是看看 SD 是怎么做的吧。


## SD 的实现

其实 SD 的实现还是有些复杂的，虽复杂，但有设计，有思想，很值得细细琢磨。在这里我们忽略了很多的实现细节, 只关注下载图片的处理过程。



### SDWebImageDownloaderOperation

前面我们提到下载图片是通过 `SDWebImageDownloaderOperation` 这个 NSOperation 来完成的，所以我们能想到唤起 progresssBlock 和 completionBlock 应该都是在这里完成的，想必图片的处理也是在这个类里处理的。

果不其然，通过浏览源码我们可以发现，在 NSURLSessionData 的代理方法中有疑似图片处理的代码，你可以打开这个类的在线代码来查看 [SDWebImageDownloaderOperation](https://github.com/rs/SDWebImage/blob/master/SDWebImage/SDWebImageDownloaderOperation.m)

```
// Create the image
CGImageRef partialImageRef = CGImageSourceCreateImageAtIndex(imageSource, 0, NULL);
// Workaround for iOS anamorphic image
if (partialImageRef)
{
    const size_t partialHeight = CGImageGetHeight(partialImageRef);
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    CGContextRef bmContext = CGBitmapContextCreate(NULL, width, height, 8, width * 4, colorSpace, kCGBitmapByteOrderDefault | kCGImageAlphaPremultipliedFirst);
    CGColorSpaceRelease(colorSpace);
    if (bmContext)
    {
        CGContextDrawImage(bmContext, (CGRect){.origin.x = 0.0f, .origin.y = 0.0f, .size.width = width, .size.height = partialHeight}, partialImageRef);
        CGImageRelease(partialImageRef);
        partialImageRef = CGBitmapContextCreateImage(bmContext);
        CGContextRelease(bmContext);
    }
    else
    {
        CGImageRelease(partialImageRef);
        partialImageRef = nil;
    }
}
```

在上门的代码中，我们很敏感的发现了 `CGContextDrawImage` 这个东东，会不会这段代码就是用来做性能优化的？

其实不然，上面的代码出现在下载过程中用已经下载好的图片片段来合成局部图片。之所以在这里调用 CoreGraphic 来绘制，是由于 iOS 有一个 bug，当我们下载的图片不是 PNG 格式的时候，显示的时候会拉伸，所以需要通过上面的手段来调整图片。

这里顺便学习点新东西，当我们需要生产局部图片的时候，可以通过 `ImageIO.framework` 里面提供的一些接口来完成工作：

```
// Get the total bytes downloaded
const NSUInteger totalSize = self.imageData.length;
// Update the data source, we must pass ALL the data, not just the new bytes
CGImageSourceRef imageSource = CGImageSourceCreateIncremental(NULL);
CGImageSourceUpdateData(imageSource, (__bridge  CFDataRef)self.imageData, totalSize == self.expectedSize);
CGImageRef partialImageRef = CGImageSourceCreateImageAtIndex(imageSource, 0, NULL);

```

我们通过 CGImageSource 对象来生成 CGImage，每次接受到新数据，我们都通过 CGImageSourceUpdateData 来更新 source，这里要注意的是，每次更新的时候，我们都要传递的是整个已经下载好的数据，而不是每次新下载的数据。每次我们都可以通过 CGImageSourceCreateImageAtIndex 来获得一个基于已经下载的数据的局部图片。


### UIImage+ForceDecode

继续探索，我们发现在唤醒回调之前，都会调用一个 UIImage 的拓展方法：

```
+ (UIImage *)decodedImageWithImage:(UIImage *)image
```

这个方法位于 SDWebImageDecoder 文件中，无论是从方法名还是从文件名来看，都无疑是做图片解码相关的工作了，这就是今天我们要找的东西了。看看具体实现，同样可以通过在线代码来查看 [SDWebImageDecoder](https://github.com/rs/SDWebImage/blob/master/SDWebImage/SDWebImageDecoder.m)

```
+ (UIImage *)decodedImageWithImage:(UIImage *)image
{
    if (image.images)
    {
        // Do not decode animated images
        return image;
    }

    CGImageRef imageRef = image.CGImage;
    CGSize imageSize = CGSizeMake(CGImageGetWidth(imageRef), CGImageGetHeight(imageRef));
    CGRect imageRect = (CGRect){.origin = CGPointZero, .size = imageSize};

    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    CGBitmapInfo bitmapInfo = CGImageGetBitmapInfo(imageRef);
    // It calculates the bytes-per-row based on the bitsPerComponent and width arguments.
    CGContextRef context = CGBitmapContextCreate(NULL,
                                                 imageSize.width,
                                                 imageSize.height,
                                                 CGImageGetBitsPerComponent(imageRef),
                                                 0,
                                                 colorSpace,
                                                 bitmapInfo);
    CGColorSpaceRelease(colorSpace);

    CGContextDrawImage(context, imageRect, imageRef);
    CGImageRef decompressedImageRef = CGBitmapContextCreateImage(context);
	
    CGContextRelease(context);
	
    UIImage *decompressedImage = [UIImage imageWithCGImage:decompressedImageRef scale:image.scale orientation:image.imageOrientation];
    CGImageRelease(decompressedImageRef);
    return decompressedImage;
}
```
从上面的代码可以看到，SD 新建了一个画布，将下载的图片强制绘制在了画布上，然后从绘制好的画布中获取新的图片并返回。

或许你会疑惑，这个方法的参数和返回都是 UIImage, 那跟我们不做任何处理直接返回图片有和区别呢？

区别大了！还记得文章刚开始提到的吗，iOS 中正常创建的 UIImage 并不会立即解码，当图片设置到 UIImageView 中去的时候，并且 CALayer 被提交到 GPU 前才会进行解码。这个解码过程发生在主线程，不可避免。所以如果我们不做处理直接返回图片，那么就必须在主线程来解码图片，如果图片很多，这很有可能就会造成界面的卡顿。

而如果我们通过图像绘制来画图，由于 CoreGraphic 方法通常是线程安全的，所以我们可以把解码的工作放在后台线程来完成，这样就减少了主线程的工作量，可以看到，SD 的解码也是在后台线程完成的。

以上就是 SD 里面对图片相关性能优化的关键地方，希望对大家有所启发。世人都说阅读源代码对于功力的提升是十分显著的，希望你也能多读读源代码。


