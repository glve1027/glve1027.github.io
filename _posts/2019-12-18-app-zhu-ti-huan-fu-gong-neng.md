---
layout:       post
title:        "App主题换肤功能"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Object-C
---

# 现在国内很多App，每到节假日，都会搞一些活动，这样就免不了需要一个统一管理所谓“皮肤”功能【这里主要指图片、字体、颜色等资源】，下面就是我设计的”皮肤“功能模块。

<!-- more -->

## 主题皮肤切换模块
### GCTTheme配置篇
* 调用GCTThemeManager之前需要注册配置数据, 调用的时机目前是在didFinishLaunchingWithOptions的时候 （这个后期可以修改！！）

```java
// 1. GCTThemeSwitchObject 控制开关的类，灵活控制开关就可以配置这个类 （面向协议）
GCTThemeSwitchObject *switchObj = [GCTThemeSwitchObject new];
[switchObj setGCTThemeSwitch:YES];
// 2. GCTThemeData 数据源的类，目前只读取本地数据，后期远程数据只需要配置它（面向协议）
[[GCTThemeManager sharedInstanceWithThemeData:[GCTThemeData new]] switchBaseOn:switchObj];
```

### GCTTheme使用篇
* 方法1：如果要修改的控件是UIView或者是它的子类并且这个UIView不等于nil，可以直接调用方法
```java
- (void)configSkinWithModule:(GCTThemeModuleTypes)module skinDatas:(NSDictionary<NSString *, NSString *> *)skinDatas;
```

* 方法2：如果上面的条件有其中一个不满足，可以直接调用单例GCTThemeManager
```java
// 需要配置皮肤替换功能的，图片获取需要 1. moduleName  2. imgKey
- (UIImage *)getImageByKey:(NSString *)imgKey from:(GCTThemeModuleTypes)module;
- (NSString *)getImageNameByKey:(NSString *)imgKey from:(GCTThemeModuleTypes)module;
// 需要配置皮肤替换功能的，颜色获取需要 1. moduleName  2. colorKey
- (UIColor *)getColorByKey:(NSString *)colorKey from:(GCTThemeModuleTypes)module;
// 需要配置皮肤替换功能的，文字获取需要 1. moduleName  2. titleKey
- (NSString *)getTitleByKey:(NSString *)titleKey from:(GCTThemeModuleTypes)module;
```

### 实际使用情况
* 直接配置TabBarController图片、文字。【因为tabBarItem默认是懒加载的，所以只能通过GCTThemeManager去实现】
```java
navController.tabBarItem.title = [[GCTThemeManager sharedInstance] getTitleByKey:model.titleKey from:GCTThemeModuleTypeHomeTabbar];
navController.tabBarItem.image = [UIImage imageMakeWithName:model.imageName type:@"png"];
NSString *imageName = [[GCTThemeManager sharedInstance] getImageNameByKey:model.imageSelectedKey from:GCTThemeModuleTypeHomeTabbar];
navController.tabBarItem.selectedImage = [[UIImage imageMakeWithName:imageName type:@"png"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
```

1. 在默认情况下，使用的是背景色。
2. 在春节皮肤的配置下使用的是图片 【这里可以以此类推，某种皮肤下面使用的属性1，在另一种皮肤下使用的是属性2，因为当找不到对应的属性值的时候返回nil，也就是不实现】

```java
self.tipsView.homeBG.image = [[GCTThemeManager sharedInstance] getImageByKey:@"home_wifi_scaning_unconnected" from:GCTThemeModuleTypeHomeWifi];
self.backgroundColor = [[GCTThemeManager sharedInstance] getColorByKey:@"home_wifi_found_scaning" from:GCTThemeModuleTypeHomeWifi];
```

### GCTTheme原理篇

* GCTThemeSwitchObject遵循的协议

```java
@protocol GCTThemeSwitchProtocol <NSObject>

- (void)setGCTThemeSwitch:(BOOL)isOpen;
- (BOOL)GCTThemeSwitchIsOpen;

@end
```

* 数据源GCTThemeData遵循的协议

```java
typedef enum : NSUInteger {
    GCTThemeDefaultData,
    GCTThemeFestivalData
} GCTThemeDataTypes;

NS_ASSUME_NONNULL_BEGIN

@protocol GCTThemeDataProtocol <NSObject>

// Getter Method
- (NSDictionary *)skinDataFor:(GCTThemeDataTypes)themeType;

// Setter Method
- (void)setSkinData:(NSDictionary *)data for:(GCTThemeDataTypes)themeType;

@end
```

* GCTThemeManager核心代码

```java
- (NSString *)findResourceBy:(NSString *)key from:(GCTThemeModuleTypes)module with:(NSString *)subKey {
    // 切换数据
    NSDictionary *resourcesData = self.switchIsOpen ? self.tempSkinData : self.skinData;
    NSString *moduleString = [self convertThemeType2String:module];
    if (moduleString == nil || [moduleString isEqualToString:@""] || resourcesData == nil || ![resourcesData.allKeys containsObject:moduleString]) return nil;
    NSDictionary *moduleDic = resourcesData[moduleString];
    if (![moduleDic.allKeys containsObject:subKey]) { return nil; }
    NSDictionary *Dic = moduleDic[subKey];
    if (![Dic.allKeys containsObject:key]) { return nil; }
    return Dic[key];
}

```

### 测试篇
* 配置以及Mock数据

```java
- (void)setUp {
    self.themeData = [GCTThemeData new];
    // Mock Data
    [self.themeData setSkinData:@{@"home_tabbar": @{@"images": @{@"image_home_button_selected": @"Home_Selected",
                                                                 @"image_video_button_selected": @"Provider_Selected",
                                                                 @"image_discovery_button_selected": @"Discovery_Selected",
                                                                 @"image_mine_button_selected": @"Mine_Selected"}}} for:GCTThemeDefaultData];
    [self.themeData setSkinData:@{@"home_middle": @{@"colors": @{@"color_before_background": @"#cccccc"},
                                                    @"values": @{@"value_home_title": @"测试"}
                                                    }} for:GCTThemeFestivalData];
    
    
    // Mock Switch
    self.switchObject = [GCTThemeSwitchObject new];
    [self.switchObject setGCTThemeSwitch:NO];
    
    self.manager = [GCTThemeManager sharedInstanceWithThemeData:self.themeData];
    [self.manager switchBaseOn:self.switchObject];
}

/*
 ** 测试是否为单例
 */
- (void)testGCTThemeManagerIsSingle {
    NSMutableArray *managers = [NSMutableArray array];
    
    for (int i = 0; i < 3; i++) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            GCTThemeManager *tempManager = [[GCTThemeManager alloc] init];
            [managers addObject:tempManager];
        });
    }
   
    [managers enumerateObjectsUsingBlock:^(GCTThemeManager *obj, NSUInteger idx, BOOL * _Nonnull stop) {
        XCTAssertEqual(self.manager, obj, @"GCTThemeManager is not single");
    }];
}

/*
 ** 测试开关
 */
- (void)testGCTThemeSwitch {
    XCTAssertFalse(self.manager.switchIsOpen, "default switch is closed");
    
    [self.switchObject setGCTThemeSwitch:YES];
    [self.manager switchBaseOn:self.switchObject];
    XCTAssertTrue(self.manager.switchIsOpen, "switch is open");
}

/*
 ** 初始化后数据是否正确
 */
- (void)testGCTThemeInitNoMockDataIsNotNil {
    XCTAssertNotNil(self.manager.skinData, "default data is not to be nil");
    XCTAssertTrue(self.manager.skinData.count != 0, "default data is not to be nil");
    XCTAssertNotNil(self.manager.tempSkinData, "temp data is not to be nil");
    XCTAssertTrue(self.manager.tempSkinData.count != 0, "temp data is not be nil");
}

/*
 ** 通过key来获取图片
 */
- (void)testGCTThemeGetImageByKey {
    XCTAssertNil([self.manager getImageByKey:@"" from:GCTThemeModuleTypeHomeTabbar], "key is nil image should be nil");
    XCTAssertNil([self.manager getImageByKey:@"image_home_button_selected_error" from:GCTThemeModuleTypeHomeTabbar], "key error image should be nil");
    XCTAssertNil([self.manager getImageByKey:@"image_home_button_selected" from:GCTThemeModuleTypeHomeWifi], @"type is error image should be nil");
    XCTAssertNotNil([self.manager getImageByKey:@"image_home_button_selected" from:GCTThemeModuleTypeHomeTabbar], "key and type is right and data has value, image should not be nil");
}

/*
 ** 通过key来获取图片名字
 */
- (void)testGCTThemeGetImageNameByKey {
    XCTAssertEqual([self.manager getImageNameByKey:@"image_home_button_selected" from:GCTThemeModuleTypeHomeTabbar], @"Home_Selected", "image name should be equal");
}

/*
 ** 通过key来获取颜色
 */
- (void)testGetColorByKey {
    // 从春节主题中获取
    [self.switchObject setGCTThemeSwitch:YES];
    [self.manager switchBaseOn:self.switchObject];
    XCTAssertTrue([[self.manager getColorByKey:@"color_before_background" from:GCTThemeModuleTypeHomeMiddleContent] isEqual:[UIColor colorWithHexString:@"#cccccc"]], "UIColor should be equal");
}

- (void)testGCTThemeGetTitleByKey {
    // 从春节主题中获取
    [self.switchObject setGCTThemeSwitch:YES];
    [self.manager switchBaseOn:self.switchObject];
    XCTAssertTrue([[self.manager getTitleByKey:@"value_home_title" from:GCTThemeModuleTypeHomeMiddleContent] isEqual:@"测试"], "Title should be equal");
}
```

### 思考篇

* 配置的时机，需要切换，尽量不要放到AppDelegate中，影响启动时间。
* GCTThemeManager中的开关GCTThemeSwitchObject以及数据源GCTThemeData相应的属性都是同时又Getter/Setter方法的，**数据不安全！！
！！都要通过字符串去查找，编码不是很友好！（后期会通过R.Obj去修正这个问题。）**
* 统一修改的时机每次启动只有一次，后期这里需要修改。
