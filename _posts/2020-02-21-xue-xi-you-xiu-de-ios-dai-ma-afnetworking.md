---
layout:       post
title:        "学习优秀的iOS代码(AFNetworking)"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Object-C
---

### 学习优秀的代码总是能让我们得到不同的反思，AFNetworking一直作为网络请求的标杆，很多iOS开发者不知道NSURLSession，但是他肯定知道AFNetworking。

<!-- more -->

#### AFHTTPSessionManager
> 如何在NSURL中添加/的字符：
> 添加/的原因：

1. 重定向 - 如果没有/就会重定向，这样会浪费带宽
2. 表示目录、信息的拼接
3. SEO 

```java
NSURL *url = [NSURL URLWithString:@"www.xxx.com"];
if (![[url absoluteString] hasSuffix:@"/"]) { url = [url URLByAppendingPathComponent:@""]; }
```
#### 配置文件的东西，都是统一写在父类AFURLSessionManager中

```java
// 所有的子类需要创建的时候，是这样创建的
- (instancetype)initWithBaseURL:(NSURL *)url sessionConfiguration:(NSURLSessionConfiguration *)configuration
{
    self = [super initWithSessionConfiguration:configuration]; // 统一在父类中配置
    if (!self) {
        return nil;
    }
    // ...
}
```

#### 在创建NSURLSession的时候，需要注意delegateQueue必须是串行的

> queue: An operation queue for scheduling the delegate calls and completion handlers. The queue should be a serial queue, in order to ensure the correct ordering of callbacks. If nil, the session creates a serial operation queue for performing all delegate method calls and completion handler calls.

```java
_session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
```

#### 在初始化AFURLSessionManager的时候，为了重新初始化session, 还存在一些之前的请求未完成，所以可以根据Session来获取相应的请求：

```java
[self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
    // 获取之前的请求
    for (NSURLSessionDataTask *task in dataTasks) {
        [self addDelegateForDataTask:task uploadProgress:nil downloadProgress:nil completionHandler:nil];
    }

    for (NSURLSessionUploadTask *uploadTask in uploadTasks) {
        [self addDelegateForUploadTask:uploadTask progress:nil completionHandler:nil];
    }

    for (NSURLSessionDownloadTask *downloadTask in downloadTasks) {
        [self addDelegateForDownloadTask:downloadTask progress:nil destination:nil completionHandler:nil];
    }
}];
```

#### 请求头参数的封装：只会有一份、并且是全局的、并且线程安全代码如下：

```java
static NSArray * AFHTTPRequestSerializerObservedKeyPaths() {
    static NSArray *_AFHTTPRequestSerializerObservedKeyPaths = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _AFHTTPRequestSerializerObservedKeyPaths = @[NSStringFromSelector(@selector(allowsCellularAccess)), NSStringFromSelector(@selector(cachePolicy)), NSStringFromSelector(@selector(HTTPShouldHandleCookies)), NSStringFromSelector(@selector(HTTPShouldUsePipelining)), NSStringFromSelector(@selector(networkServiceType)), NSStringFromSelector(@selector(timeoutInterval))];
    });

    return _AFHTTPRequestSerializerObservedKeyPaths;
}
```

#### 关闭自动KVO，手动触发

> 这里关于如何在配置选项中设置值的操作还是挺不错的，运用响应式的操作模式。【具体可以详见类AFHTTPRequestSerializer中的第377行左右，运用KVO设置属性值的操作】

```java
// 关闭KVO
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key {
    if ([AFHTTPRequestSerializerObservedKeyPaths() containsObject:key] {
        return NO;
    }

    return [super automaticallyNotifiesObserversForKey:key];
}

// 手动触发KVO
- (void)setCachePolicy:(NSURLRequestCachePolicy)cachePolicy {
    [self willChangeValueForKey:NSStringFromSelector(@selector(cachePolicy))];
    _cachePolicy = cachePolicy;
    [self didChangeValueForKey:NSStringFromSelector(@selector(cachePolicy))];
}
```


#### 关于如何将NSDictionary的数据结构转化为GET请求中需要拼接在URL中/POST请求中需要将请求放入的请求体中
> 核心的将NSDictionary转化为NSArray的数据的函数：

```java

// 上面的函数中运用了递归的方式来处理参数，有以下几点是亮点：

// 1. 传入NSDictionary首先需要对key按照ASCII值进行排序，作者使用了NSSortDescriptor, 关于这个类的使用我下面给出一个范例。
// 2. 具体讲数组中的对象拼接城NSString *的时候，核心的处理转译等细节处理放在函数AFPercentEscapedStringFromString中 【运用这点功能，其实我们就能实现一个自定义的路由功能】
// 3. 这一点我有点没有想明白，作者为什么这里大量运用了C语言函数而不是OC函数 【！！不解】
// 4. 关于URL对象中是否存在?，这个细节将决定我们改如何拼接之前已经处理好的参数，这里如果是我来处理的话，就会直接通过字符串的功能，直接查找的?这个符号，但是作者却没有这样使用，作者是这样的判断的：

// mutableRequest.URL.query ? @"&%@" : @"?%@", query
// 关于query的解释如下：This property contains the query string. Any percent-encoded characters are not unescaped. If the receiver does not conform to RFC 1808, this property contains nil. For example, in the URL http://www.example.com/index.php?key1=value1&key2=value2, the query string is key1=value1&key2=value2.

NSArray * AFQueryStringPairsFromKeyAndValue(NSString *key, id value) {
    
}
```

#### 关于NSSortDescriptor的使用，我写了个Demo:

```java
// Person.h
@interface Person : NSObject
@property (strong, nonatomic) Salary *salary;
@property (copy, nonatomic) NSString *firstName;
@property (copy, nonatomic) NSString *lastName;
+ (instancetype)initWithFirstName:(NSString *)first lastName:(NSString *)last;
+ (instancetype)initWithFirstName:(NSString *)first lastName:(NSString *)last salary:(NSInteger)salary;
@end

// Person.m
@implementation Person
+ (instancetype)initWithFirstName:(NSString *)first lastName:(NSString *)last {
    Person *p = [Person new];
    p.firstName = first;
    p.lastName  = last;
    return p;
}

+ (instancetype)initWithFirstName:(NSString *)first lastName:(NSString *)last salary:(NSInteger)salary {
    Person *p = [Person initWithFirstName:first lastName:last];
    if (p != nil) {
        Salary *s = [Salary new];
        s.amount = salary;
        p.salary = s;
    }
    return p;
}

- (NSString *)description {
    return self.salary == nil ? [NSString stringWithFormat:@"firstName:%@-lastName:%@", self.firstName, self.lastName] : [NSString stringWithFormat:@"firstName:%@-lastName:%@-salary:%ld", self.firstName, self.lastName, (long)self.salary.amount];
}
@end

// Salary.h
@interface Salary : NSObject
@property (nonatomic, assign) NSInteger amount;
@end

// Test
NSMutableArray <Person *> *persons = [NSMutableArray new];
[persons addObject:[Person initWithFirstName:@"1" lastName:@"100"]];
[persons addObject:[Person initWithFirstName:@"2" lastName:@"99"]];
[persons addObject:[Person initWithFirstName:@"98" lastName:@"3"]];
[persons addObject:[Person initWithFirstName:@"98" lastName:@"4"]];
[persons addObject:[Person initWithFirstName:@"97`" lastName:@"4"]];
[persons addObject:[Person initWithFirstName:@"96`" lastName:@"4" salary:1]];
[persons addObject:[Person initWithFirstName:@"96`" lastName:@"4" salary:2]];

// 创建排序条件
NSSortDescriptor *sd1 = [[NSSortDescriptor alloc] initWithKey:@"firstName" ascending:YES];
NSSortDescriptor *sd2 = [[NSSortDescriptor alloc] initWithKey:@"lastName" ascending:NO];
NSSortDescriptor *sd3 = [[NSSortDescriptor alloc] initWithKey:@"salary.amount" ascending:NO]; // 这里的key, 类似于KVO中的键值d观察，可以具体观察某个对象的属性值

NSArray *sortedDatas = [persons sortedArrayUsingDescriptors:@[sd1, sd2, sd3]];

for (id data in sortedDatas) {
    NSLog(@"%@", data);
}
```

#### 通过NSURLSession和NSURLRequest来创建NSURLSessionDataTask的过程，AFNetworking做了一个版本兼容的处理，操作如下：

```java

/**
因为AFNetworking的taskIdentifier和回调代理一一绑定的
首先这里创建队列以及操作的函数作者还是封装成了static的函数
可以看到这里是为了处理在iOS8以下版本下的一个bug，具体解释可以看这个[issue](https://github.com/AFNetworking/AFNetworking/issues/2093),解释如下：
Summary:
taskIdentifiers on an NSURLSessionDataTask should be unique per NSURLSession, as the documentation states. The attached projects demonstrates an issue where tasks that are created concurrently can end up with the same taskIdentifier.
Steps to Reproduce:

Dispatch 100 data tasks on the default priority queue.
Monitor the taskIdentifiers returned by the task.
Observe that they are occasionally the same — likely, since the queue is concurrent, this is when two tasks end up getting created on different threads around the same time.
Expected Results:
Task Identifiers should be unique per NSURLSession, as the documentation states.
Actual Results:
Task Identifiers are not always unique. The popular AFNetworking framework depends on this uniqueness, and can end up calling incorrect completionHandlers.
Version:
iOS 7.1.1
简单来说就是在通过并行的方式来创建task的时候，会偶尔出现taskIdentifier和之前的一模一样，这样就会导致之前的回调如果还没有回调回来，就会将之前那个回调强制remove掉
*/

__block NSURLSessionDataTask *dataTask = nil;
url_session_manager_create_task_safely(^{
    dataTask = [self.session dataTaskWithRequest:request];
});

static void url_session_manager_create_task_safely(dispatch_block_t _Nonnull block) {
    if (block != NULL) {
        if (NSFoundationVersionNumber < NSFoundationVersionNumber_With_Fixed_5871104061079552_bug) {
            // Fix of bug
            // Open Radar:http://openradar.appspot.com/radar?id=5871104061079552 (status: Fixed in iOS8)
            // Issue about:https://github.com/AFNetworking/AFNetworking/issues/2093
            dispatch_sync(url_session_manager_creation_queue(), block);
        } else {
            block();
        }
    }
}

static dispatch_queue_t url_session_manager_creation_queue() {
    static dispatch_queue_t af_url_session_manager_creation_queue;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        af_url_session_manager_creation_queue = dispatch_queue_create("com.alamofire.networking.session.manager.creation", DISPATCH_QUEUE_SERIAL);
    });

    return af_url_session_manager_creation_queue;
}
```

```java
// 回调代理AFURLSessionManagerTaskDelegate和taskNSURLSessionTask形成绑定关系，但是这里有一点不是很确定【！！不确定，只是自己的猜测】
// 作者这里并没有直接监听task对象的countOfBytesReceived/countOfBytesSent/countOfBytesExpectedToSend等等【我看之前有个版本的AF是直接监听task的这些属性值】，而是现在是直接创建了一个NSProgress对象，然后每次在代理回调的时候，赋值。
// 这里猜测，是因为有可能上传文件的totalUnitCount会一直发生变化，不是说起初设置了之后就不会再发生变化了。关于如何使用NSProgess,我写了个Demo,如下:

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.progress = [[NSProgress alloc] initWithParent:nil userInfo:nil];
    self.progress.totalUnitCount = 100;
    [self.progress addObserver:self forKeyPath:NSStringFromSelector(@selector(fractionCompleted)) options:NSKeyValueObservingOptionNew context:nil];
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    //self.progress.completedUnitCount += 10;// 这里就算超出了也没有问题，但是这里最好不要这么做，做一下判断
    if (self.progress.completedUnitCount < self.progress.totalUnitCount) {
        self.progress.completedUnitCount = (self.progress.completedUnitCount + 10) >= self.progress.totalUnitCount
                                            ? self.progress.totalUnitCount
                                            : self.progress.completedUnitCount + 10;
    }
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    if ([keyPath isEqualToString:NSStringFromSelector(@selector(fractionCompleted))]) {
        NSLog(@"--->%@", object);
    }
}
```

#### AFNetworking如何实现一个简单的工作流

> 这里所谓的工作流,只是简单地顺序流程，例如：流程A做完->流程B做完->流程C做完
> 具体的操作就是类NSInputStream流的操作：【具体参照AFNetworking多表单的发送中类AFHTTPBodyPart的操作】

```java
- (BOOL)transitionToNextPhase {
    // 保证在主线程操作
    if (![[NSThread currentThread] isMainThread]) {
        dispatch_sync(dispatch_get_main_queue(), ^{
            [self transitionToNextPhase];
        });
        return YES;
    }

    switch (_phase) {
        case AFEncapsulationBoundaryPhase:
            _phase = AFHeaderPhase; // 跳转的下一个流程
            break;
        case AFHeaderPhase:
            [self.inputStream scheduleInRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
            [self.inputStream open];
            _phase = AFBodyPhase;  // 跳转的下一个流程
            break;
        case AFBodyPhase:
            [self.inputStream close];
            _phase = AFFinalBoundaryPhase;  // 跳转的下一个流程
            break;
        case AFFinalBoundaryPhase:
        default:
            _phase = AFEncapsulationBoundaryPhase;  // 跳转第一个流程，准备下一次开始流程
            break;
    }
    _phaseReadOffset = 0;

    return YES;
}

// 那么关键来了，在什么时候，需要调用这个方法呢？
// 1. 初始化的时候
- (instancetype)init {
    //...xxx
    [self transitionToNextPhase];
    //...xxx
}

// 2. 关闭、错误的时候需要扭转状态
// 这里的代码写的很巧妙
// 枚举值如下
/*
typedef NS_ENUM(NSUInteger, NSStreamStatus) {
    NSStreamStatusNotOpen = 0,
    NSStreamStatusOpening = 1,
    NSStreamStatusOpen = 2,
    NSStreamStatusReading = 3,
    NSStreamStatusWriting = 4,
    NSStreamStatusAtEnd = 5,
    NSStreamStatusClosed = 6,
    NSStreamStatusError = 7
};
*/
if ([self.inputStream streamStatus] >= NSStreamStatusAtEnd) {
    [self transitionToNextPhase];
}

// 3. 正常读完数据的时候需要扭转状态
if (((NSUInteger)_phaseReadOffset) >= [data length]) {
    [self transitionToNextPhase];
}

// 上面的扭转，写的是很巧妙，但是却不适用很复杂，多情况的扭转，类似状态机，但是还是值得借鉴。
```

#### 如何通过多表单上传信息

```java
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
manager.requestSerializer.timeoutInterval = 5;
manager.requestSerializer.allowsCellularAccess = 1;
AFJSONResponseSerializer *ser = [AFJSONResponseSerializer serializer];
manager.responseSerializer = ser;
ser.removesKeysWithNullValues = YES;// 遇到空值是否移除 null
NSDictionary *dic = @{ @"name": @"gh", @"age": @(12), @"info": @{ @"address": @"hangzhou" }};
[manager POST:@"https://www.baidu.com" parameters:dic headers:@{} constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
    /*
    // 在block中设置需要上传的文件
    NSString *path = [[NSBundle mainBundle] pathForResource:@"1" ofType:@"png"];
    // 将本地的数据拼接到formData中，指定name
    [formData appendPartWithFileURL:[NSURL fileURLWithPath:path] name:@"file" error:nil];
    
    // 或者使用这个接口拼接 指定name和filename
    NSData *picData = [NSData dataWithContentsOfFile:path];
    [formData appendPartWithFileData:picData name:@"image" fileName:@"image.jpg" mimeType:@"image/jpeg"];
    */
} progress:^(NSProgress * _Nonnull uploadProgress) {
        
} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
    
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
    
}];
```