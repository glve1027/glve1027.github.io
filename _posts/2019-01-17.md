---
layout:       post
title:        "关于Kiwi测试框架"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Object-C
---

#### 安装: 通过`Cocoapods`安装`Kiwi`，命令如下：
```shell
target 'xxxTests' do
  inherit! :search_paths
  # Pods for testing
  pod 'Kiwi'
end
```
<!-- more -->

#### 安装`Kiwi`的`Xcode Template`
1. 下载 [Kiwi Template](https://github.com/kiwi-bdd/Kiwi/tree/master/Xcode%20Templates)
2. 执行安装脚本 `sh ./install-templates.sh`

#### 创建
1. 通过`Xcode`-> `File` -> `New` -> `File...`
2. 输入`Kiwi`过滤出`Kiwi Spec` -> `Next` 创建完成 [ 这里输入的文件名就是你要测试的类名 ]

#### 使用
1. Kiwi测试中的行为描述：`Given`、`When`、`Then`
2. 一个测试类中，一个测试类只会存在一个`describe` (一个测试文件应该专注于一个测试类)
3. 一个`describe`可以存在多个`context`
4. 一个`context`可以存在多个`it`

#### 这里写了一个例子，一个简单的User对象, [基本断言测试文档](https://github.com/kiwi-bdd/Kiwi/wiki/Expectations)

```java
#import <Foundation/Foundation.h>

@class DataBaseManager;

NS_ASSUME_NONNULL_BEGIN

@interface GCTUser : NSObject

@property (nonatomic, copy, readonly) NSString *userName;
@property (nonatomic, assign, readonly) NSInteger password;
@property (nonatomic, strong, readonly) NSArray *addressDatas;

- (instancetype)initWithUserName:(NSString *)userName password:(NSInteger)password;

- (void)registerAddressWith:(NSString *)addressName;
- (void)unregisterAddress;
// 测试Exception断言
- (void)forceUnregisterAddress;

// 获取用户信息数据
// 1. 
- (NSDictionary *)userInfos;
// 2. 外部传入
- (NSDictionary *)getUserInfosFrom:(DataBaseManager *)dataBaseManager;
@end

NS_ASSUME_NONNULL_END
```

```java
describe(@"Given a GCTUser", ^{ // <--Given
    /* -- 1 -- */
    context(@"when user userName is 'Hello world' and password is '123456'", ^{// <-- When
        
        GCTUser *user = [[GCTUser alloc] initWithUserName:@"Hello world" password:123456];
        
        it(@"user is exists", ^{
            [[user shouldNot] beNil];
        });
        
        it(@"userName is 'Hello world'", ^{ // <-- Then (with description)
            [[user.userName should] equal:@"Hello world"];
        });
        
        specify(^{
            [[theValue(123456) should] equal:theValue(user.password)];  // <-- Then (without description)
        });
    });
    
    /* -- 2 -- */
    context(@"when user created", ^{// <-- When
       
        __block GCTUser *user = nil;
        beforeAll(^{
            user = [GCTUser new];
        });
        
        afterAll(^{
            user = nil;
        });
        
        it(@"when user is created, address data should be empty", ^{ // <-- Then (with description)
            [[user.addressDatas should] beEmpty];
        });
    });
    
    /* -- 3 -- */
    context(@"when register address name", ^{// <-- When
        GCTUser *user = [GCTUser new];
        beforeEach(^{
            [user registerAddressWith:@"hangzhou"];
        });
        
        afterEach(^{
            [user unregisterAddress];
        });
        
        it(@"after user register, then", ^{ // <-- Then (with description)
            [[user.addressDatas.firstObject should] equal:@"hangzhou"];
        });
    });
    
    /* -- 4 -- */
    context(@"when unregister address", ^{// <-- When
        GCTUser *user = [GCTUser new];
        beforeAll(^{
            [user registerAddressWith:@"beijing"];
        });
        
        beforeEach(^{
            [user unregisterAddress];
        });
        
        it(@"after user unregister", ^{// <-- Then (with description)
            [[user.addressDatas should] beEmpty];
        });
    });
    
    /* -- 5 -- */
    context(@"when user addresses are empty", ^{
        GCTUser *user = [GCTUser new];
        
        it(@"called force ungister, will throw exception", ^{
            [[theBlock(^{
                [user forceUnregisterAddress];
            }) should] raiseWithName:@"GCTUserAddressDatasEmptyException"] ;
        });
    });
});
```
#### 第二个例子，我用MVVM模式，写了一个简单的页面，这里面用到了UITableView, 重点测试ViewModel, viewModel代码如下：

```java
#import <UIKit/UIKit.h>
#import "HttpClient.h"

typedef void(^TableViewCellConfigBlock)(id cell, id item);
typedef void(^HomeViewModelSuccessBlock)(void);
typedef void(^HomeViewModelFailureBlock)(NSError *e);

NS_ASSUME_NONNULL_BEGIN

@interface HomeViewModel : NSObject<UITableViewDataSource>
//**??**
@property (strong, nonatomic) HttpClient *httpClient; // 具体实现网络请求的类

// 实例化方法
- (instancetype)initWithCellIdentifior:(NSString *)cellIdentifior configureCellBlock:(TableViewCellConfigBlock)configureCellBlock;

// 网络请求方法
- (void)requestHomeDatasBasedOn:(NSString *)homeID
                        success:(HomeViewModelSuccessBlock)success
                        failure:(HomeViewModelFailureBlock)failure;

@end

NS_ASSUME_NONNULL_END
```

#### 测试网络请求类：
> 这里主要用到了异步断言[[expectFutureValue(theValue(errorOccur)) shouldEventuallyBeforeTimingOutAfter(2.0)] equal:theValue(YES)]; [Asynchronous测试参考文档](https://github.com/kiwi-bdd/Kiwi/wiki/Asynchronous-Testing)

```java
describe(@"HomeViewModel", ^{

    context(@"Normal Request", ^{

        HomeViewModel *viewModel = [HomeViewModel new];
        __block NSError *noInputError = nil;
        __block BOOL errorOccur = NO;
        __block BOOL successOccur = NO;

        it(@"当入参HomeID为空的时候，将会调用failure的callback, 并且会返回制定的error", ^{
            [viewModel requestHomeDatasBasedOn:@"" success:^{} failure:^(NSError * _Nonnull error) {
                noInputError = error;
                errorOccur = YES;
            }];
            [[expectFutureValue(theValue(errorOccur)) shouldEventuallyBeforeTimingOutAfter(2.0)] equal:theValue(YES)];
            [[noInputError shouldNot] beNil];
            [[noInputError should] equal:[NSError errorWithDomain:@"empty input" code:1011 userInfo:nil]];
        });


        it(@"当传入HomeID的时候，就会调用success的callBack, 并且会返回数据", ^{
            [viewModel requestHomeDatasBasedOn:@"HomeID" success:^{
                successOccur = YES;
            } failure:^(NSError * _Nonnull error) {}];
            [[expectFutureValue(theValue(successOccur)) shouldEventuallyBeforeTimingOutAfter(2.0)] equal:theValue(YES)];
        });
    });
});
```

#### 这里我可以通过Mock HttpClient的返回数据来测试返回结果

```java
 context(@"Mock Request", ^{
        
        HomeViewModel *homeViewModel = [HomeViewModel new];
        id mockTableView = [UITableView mock];
        
        it(@"如果网络请求返回的是两个数据，则UITableView DataSource的数据源也是两个", ^{
            HttpClient *client = [HttpClient new];
            NSArray *mockDatas = @[[[HomeModel alloc] initWithTitle:@"test-1"], [[HomeModel alloc] initWithTitle:@"test-2"], [[HomeModel alloc] initWithTitle:@"test-2"]];
            [client stub:@selector(requestHomeDataBy:success:failure:) withBlock:^id(NSArray *params) {
                HttpClientSuccessBlock successB = [params objectAtIndex:1];
                successB(mockDatas);
                return nil;
            }];
            
            
            homeViewModel.httpClient = client;
            [homeViewModel requestHomeDatasBasedOn:@"test" success:^{
                NSInteger count = [homeViewModel tableView:mockTableView numberOfRowsInSection:0];
                [[theValue(count) should] equal:theValue(3)];
                
                
                
            } failure:^(NSError *e) {}];
        });
});
```

#### 测试block的回调代码如下，和上面的mock httpClient雷同
```java
context(@"Configuration", ^{
       __block UITableViewCell *configuredCell = nil;
       __block id configuredObject = nil;
       
       TableViewCellConfigBlock block = ^(UITableViewCell *a, id b){
           configuredCell = a;
           configuredObject = b;
       };
       HomeViewModel *homeViewModel = [[HomeViewModel alloc] initWithCellIdentifior:@"HomeTableViewCell" configureCellBlock:block];
       
       id mockTableView = [UITableView mock];
       UITableViewCell *cell = [[UITableViewCell alloc] init];
       
       __block id result = nil;
       NSIndexPath *indexPath = [NSIndexPath indexPathForRow:0 inSection:0];
       
       it(@"如果网络请求返回的是两个数据，则UITableView DataSource的数据源也是两个", ^{
           HttpClient *client = [HttpClient new];
           NSArray *mockDatas = @[[[HomeModel alloc] initWithTitle:@"test-1"], [[HomeModel alloc] initWithTitle:@"test-2"], [[HomeModel alloc] initWithTitle:@"test-2"]];
           [client stub:@selector(requestHomeDataBy:success:failure:) withBlock:^id(NSArray *params) {
               HttpClientSuccessBlock successB = [params objectAtIndex:1];
               successB(mockDatas);
               return nil;
           }];
           
           
           homeViewModel.httpClient = client;
           [homeViewModel requestHomeDatasBasedOn:@"test" success:^{
               
               [[mockTableView should] receive:@selector(dequeueReusableCellWithIdentifier:forIndexPath:) andReturn:cell withArguments:@"HomeTableViewCell",indexPath];
               result = [homeViewModel tableView:mockTableView cellForRowAtIndexPath:indexPath];
               
               [[result should] equal:cell];

               [[configuredCell should] equal:cell];

               [[configuredObject should] beKindOfClass:[HomeModel class]];
               HomeModel *hM = (HomeModel *)configuredObject;
               [[hM.showTitle should] equal:@"test-1"];
               
           } failure:^(NSError *e) {}];
       });
   });
```
* 这里有点问题：
1. 首先，测试结果仍旧取决于网络状况，因此我们很难保证多次测试结果的一致性；
2. 其次，当我们要测试一个REST服务的时候，如果每个URL的测试都基于实际网络访问和超时的机制，将会显著增加测试执行的时间
> 我的观点是 1. 测试过程要不依赖于任何外部条件和系统；2. 在任何环境、测试任意多次，结果应该保持不变；

#### 页面的跳转测试
> 这里主要用到了`stub`和`mock`
> 维基百科对stub的解释 : 桩[1]（Stub / Method Stub）是指用来替换一部分功能的程序段。桩程序可以用来模拟已有程序的行为（比如一个远端机器的过程）或是对将要开发的代码的一种临时替代。因此，打桩技术在程序移植、分布式计算、通用软件开发和测试中用处很大
> 维基百科对mock的解释 : 在面向对象程序设计中，模拟对象（英语：mock object，也译作模仿对象）是以可控的方式模拟真实对象行为的假的对象。程序员通常创造模拟对象来测试其他对象的行为，很类似汽车设计者使用碰撞测试假人来模拟车辆碰撞中人的动态行为。

* 我的理解是这样的，如果你只是需要仿照掉一个方法，就用stub，如果需要仿照的是一个对象，就用mock

```java
describe(@"HomeViewController", ^{
    context(@"when click a cell in table view", ^{
        it(@"A PushViewController should be pushed", ^{
            HomeViewController *homeViewController = [[HomeViewController alloc] init];
            UIView *view = homeViewController.view;
            [[view shouldNot] beNil];
            
            UINavigationController *mockNavController = [UINavigationController mock];
            [homeViewController stub:@selector(navigationController) andReturn:mockNavController];
            
            [[mockNavController should] receive:@selector(pushViewController:animated:)];
            KWCaptureSpy *spy = [mockNavController captureArgument:@selector(pushViewController:animated:) atIndex:0];
            [homeViewController tableView:homeViewController.tableView didSelectRowAtIndexPath:[NSIndexPath indexPathForRow:0 inSection:0]];
            
            id obj = spy.argument;
            PushViewController *vc = obj;
            [[vc should] beKindOfClass:[PushViewController class]];
        });
    });
});
```
