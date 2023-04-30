---
layout:       post
title:        "读《Effective Objective-C2.0》的一些思考🤔"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Object-C
---

### 闲来无事，最近没事会看看书，其实书有很多，但是能称得上好书却不是很多：啥事好书呢？
> 我认为，好书能够给不同层次的人给予思考。因为这样思考出来的东西，就不仅仅是书面知识所传授的，

* 下面会列举一下好书中好的东西【1. 当然这一点的理解很主观 2. 好书里面也会存在冗余】,以及一些我个人的思考🤔
* 正常些OC的人肯定听过《Effective Objective-C2.0》【这本书里70-80%还都是不错的】

#### 关于第九点：类族模式
> 我不是很建议这样去写一个类，但让可以换个方式，通过接口，而不是硬生生通过一个抽象类，并且生成一个子类的方式，这本的书的作者举的例子我觉得不是很好，如果用我可能会通过定义接口的形式来去做，但是这样的话，就超出这本的范畴。

#### 关于第十二点：理解消息转发机制
> 这事一个很入门，很具体有意义的例子，建议手敲一遍：

```java
id autoDictionaryGetter(id self, SEL _cmd) {
    // Get the backing stroe from the object
    EOCAutoDictionary *typeSelf = (EOCAutoDictionary *)self;
    NSMutableDictionary *backingStroe = typeSelf.backingStroe;
    
    // The key is simple the selector name
    NSString *key = NSStringFromSelector(_cmd);
    
    // Return the value
    return [backingStroe objectForKey:key];
}

void autoDictionarySetter(id self, SEL _cmd, id value) {
    // Get the backing store from the object
    EOCAutoDictionary *typeSelf = (EOCAutoDictionary *)self;
    NSMutableDictionary *backingStroe = typeSelf.backingStroe;
    
    /** The selector will be for example: "setOpaqueObject:".
     *  We need to remove the "set", ":" and lowercase the first
     *  letter of the remainder
     */
    NSString *selectorString = NSStringFromSelector(_cmd);
    NSMutableString *key = [selectorString mutableCopy];
    
    // Remove the ":" at the end
    [key deleteCharactersInRange:NSMakeRange(key.length - 1, 1)];
    
    // Remove the 'set' prefix
    [key deleteCharactersInRange:NSMakeRange(0, 3)];
    
    // Lowercase the first character
    NSString *lowercaseFirstChar = [[key substringToIndex:1] lowercaseString];
    [key replaceCharactersInRange:NSMakeRange(0, 1) withString:lowercaseFirstChar];
    
    if (value) {
        [backingStroe setObject:value forKey:key];
    } else {
        [backingStroe removeObjectForKey:key];
    }
}

@dynamic string, number, date, opaqueObject;

- (instancetype)init {
    if (self = [super init]) {
        _backingStroe = [NSMutableDictionary new];
    }
    return self;
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selectorString = NSStringFromSelector(sel);
    if ([selectorString hasPrefix:@"set"]) {
        class_addMethod(self, sel, (IMP)autoDictionarySetter, "v:@:@");
    } else {
        class_addMethod(self, sel, (IMP)autoDictionaryGetter, "@@:");
    }
    return YES;
}
```

#### 第15点：用前缀避免命名空间冲突
> 这一点表面看上去学不到什么知识点，但是你可以看一下AFNetworking源码，你就会发现作者框架内部的很多静态的C函数，定义是完全符合这一点的、包括一些常量的定义【queue的定义】，从这一点反向来证明使用前缀的重要性。

#### 第16点：提供"全能初始化方法"
> 这一条，看标题很唬人，其实作为以面向对象的代码而言，不仅仅初始化需要这样做，我们所有的函数，都应该这样做，抽象。【PS: 这一点特性，可以看到SDWebImage的库、AFNetworking的库到处都有这样的操作，值得学习】

#### 第17点： 实现description方法
> 这一点其实说实话，不是很有用，因为现在的ide已经很强大了，根本不需要通过打印信息，你完全可以通过LLDB来调试 【冷门：LLDB命令po出的信息，是debugDescription所打印的，当然它默认调用的是description知道就行】

#### 第18点：很关键，这一点可以简化代码复杂度、减少产生bug的几率
> 因为一个面向很多人SDK, 你可能需要的是每个属性、每个函数是否都是线程安全的，会不会出现异常？作者提出这一点很好的规避了这一点，属性只读，不要把可变的collection作为属性公开、经量创建不可变对象。【如果你一定要挑战自己，你就把属性都暴露出去，看看你锁、线程用的6不？】

#### 第19点：使用清晰而协调的命名方式
> 说一个现象，从Swift 1.0 -> Swift 5.0 被开发者不断的吐槽一个点，就一个函数名，可能会变好几次，之前版本作废的函数名，在下一个版本又出现了，足以见得，取函数名是程序员最头痛的事情。【OC这样的命名方式不能说很完美，因为Swift以及现在很多主流的语言都在往一个方向房展，一个唯一的标准：“言简意赅”】

#### 第20点：为私有方法名加前缀
> 这一点你就听你们组、以及自己公司的代码规范就行了，实在不行可以规范一套

#### 第21点：理解Object-C错误模型
> 现有的框架库还都是沿用C语言的这一套，运用指针的方式来实现，但是你完全可以用block来实现，下面的代码可以学习一下，设计SDK的时候，也可以这么使用：

```java
// EOCErrors.h
extern NSString * const EOErrorDomain;

typedef NS_ENUM(NSUInteger, EOCError) {
    EOCErrorUnknow = -1,
    EOCErrorInternalInconsistency = 100,
    EOCErrorGeneralFault = 105,
    EOCErrorBadInput = 500
};

// EOCErrors.m
NSString *const EOErrorDomain = @"EOCErrorDomain";
```

#### 第22点：理解NSCoping协议
> 这里关于深/浅拷贝没什么可说的，但是例子中提到的另一个问题值得注意，如果某个属性值并非属性，只是个在内部使用的实例变量，可以通过->来访问

```java
- (id)copyWithZone:(NSZone *)zone {
    EOCAutoDictionary *copy = [[self class] allocWithZone:zone];
    copy -> friends = [_friends copy];
    return copy;
}
```