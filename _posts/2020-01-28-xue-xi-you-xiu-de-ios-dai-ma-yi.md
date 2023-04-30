---
layout:       post
title:        "学习优秀的iOS代码(SDWebImage)"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Object-C
---

### [SDWebImage](https://github.com/SDWebImage/SDWebImage),一个iOS图片加载库，在iOS界，一直是标杆的存在，本篇博客不去讨论他的具体代码架构、设计、算法，而是从一个个小的tips来发现这些代码的闪光点。

<!-- more -->

#### 1. 协议(接口)的使用
> 自吹一下，作者的这一点写代码的风格和我还是很像的，都很喜欢面向接口编程，整个SDWebImage中存在很多这样的情况，比如：SDImageCodersManager， 光从这个名字就能看出这是一个关于给图片编码的类，再看定义：

```java
@interface SDImageCodersManager : NSObject <SDImageCoder>
@end

@protocol SDImageCoder <NSObject>
@end
```

> 2. 这样代码风格遍布在整个项目中, 例如下面类中定义的SDWebImageDownloaderRequestModifier,SDWebImageDownloaderResponseModifier,SDWebImageDownloaderDecryptor

```java
@property (nonatomic, strong, nullable) id<SDWebImageDownloaderRequestModifier> requestModifier;

@property (nonatomic, strong, nullable) id<SDWebImageDownloaderResponseModifier> responseModifier;

@property (nonatomic, strong, nullable) id<SDWebImageDownloaderDecryptor> decryptor;
```

#### 总结：如果后面我们也想实现一个功能可以模仿这种规范：1：定义一个接口 2：定义一个默认的类并且遵循相应的接口 3：暴露给外面的是一个id<协议>object。

#### 2. 宏的使用
> 其实如果你正常看第三个优秀源码，你就会发现，很多代码存在很多优秀的宏定义，一方面使用起来很优雅，一方面也可以学习到关于编译器的一个参数的设置。在项目中，我经常看到下面的代码：

```java
SD_LOCK(self.HTTPHeadersLock);
[self.HTTPHeaders setValue:value forKey:field];
SD_UNLOCK(self.HTTPHeadersLock);

// 定义
#ifndef SD_LOCK
#define SD_LOCK(lock) dispatch_semaphore_wait(lock, DISPATCH_TIME_FOREVER);
#endif

#ifndef SD_UNLOCK
#define SD_UNLOCK(lock) dispatch_semaphore_signal(lock);
#endif

// 用来区分平台的宏
#if TARGET_OS_IOS || TARGET_OS_TV
    #define SD_UIKIT 1
#else
    #define SD_UIKIT 0
#endif

#if TARGET_OS_IOS
    #define SD_IOS 1
#else
    #define SD_IOS 0
#endif
```

#### 总结：没什么技术含量，大家都会这样去实现锁,但是代码未必有这么优雅，点赞。
说道锁的实现，代码中还存在另一个保证线程安全的就是@synchronized(self), 这里在项目源码中，有时候作者用的是@synchronized(self), 有时候用的又是一个临时变量：@synchronized(xx), 网上之前也有一些关于这样性能不是很好的传闻。关于这个我特意去查阅了一些资料，大家可以看一下[FaceBook大神的这篇文章](http://mrpeak.cn/blog/synchronized/)

#### 3. 关于函数定义规范的问题
> 在项目中关于下载图片的api，可以说是核心的代码，在看看内部函数定义，代码如下：

```java
// 参数类型的定义
typedef void(^SDWebImageNoParamsBlock)(void);
typedef NSString * SDWebImageContextOption NS_EXTENSIBLE_STRING_ENUM;
typedef NSDictionary<SDWebImageContextOption, id> SDWebImageContext;
typedef NSMutableDictionary<SDWebImageContextOption, id> SDWebImageMutableContext;


- (void)sd_setImageWithURL:(nullable NSURL *)url {
    [self sd_setImageWithURL:url placeholderImage:nil options:0 progress:nil completed:nil];
}

- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder {
    [self sd_setImageWithURL:url placeholderImage:placeholder options:0 progress:nil completed:nil];
}

- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder options:(SDWebImageOptions)options {
    [self sd_setImageWithURL:url placeholderImage:placeholder options:options progress:nil completed:nil];
}

- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder options:(SDWebImageOptions)options context:(nullable SDWebImageContext *)context
```
* 后期定义函数也可以这样，最后收拢到一个核心的函数上。
* 关于参数的类型，是否需要提前定义，这个从规整的角度来看，还是需要的。
* 关于函数类型是否为nil，还是需要nullable来标识。
> 越是规范的东西，越是在写代码的时候可以的去实现，时间长了，也可以学习到优秀代码的规范。

#### 4. 关于网络请求的实现

* 这里用自定义的NSOperation, 这里要说一下，自定义NSOperation有两种实现，一种是从写start方法【需要自己去管理isFinish/isExecuting】, 领一种是从写main方法【不需要管理状态值】
* 并且这里上来就用了@synchronized保证线程安全。
* 并且这里使用了一个自定义的串行队列来管理_coderQueue = dispatch_queue_create("com.hackemist.SDWebImageDownloaderOperationCoderQueue", DISPATCH_QUEUE_SERIAL);

```java
- (void)start {
    @synchronized (self) {
        ...
    }
}
```

#### 5. 关于代码设计：开闭原则
> 其实在设计Framework的时候，需要时刻记着，哪些东西不应该暴露给用户，哪些需要暴露给用户修改的，这里作者大量使用枚举来实现外部自定义配置：

```java
if (self.options & SDWebImageDownloaderHighPriority) {
    self.dataTask.priority = NSURLSessionTaskPriorityHigh;
} else if (self.options & SDWebImageDownloaderLowPriority) {
    self.dataTask.priority = NSURLSessionTaskPriorityLow;
}      
```



