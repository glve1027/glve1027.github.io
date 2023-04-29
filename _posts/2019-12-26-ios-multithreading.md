---
layout:       post
title:        "iOS多线程操作"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Object-C
---

> 一旦项目中涉及到多线程操作，就会显得异常复杂，尤其是对数据的操作，如何保证高效的“多读单写”呢？

# 方案有很多种：

* 可以使用锁[这里就不具体展开了，锁的实现太多了]，这里重点说一下第二个方案：
* 利用GCD实现“多读单写”的功能

```objc
// 创建一个全局的自定义并行队列
dispatch_queue_t globalQueue = dispatch_queue_create("xxx", DISPATCH_QUEUE_CONCURRENT);
// 对于"写"的操作
// 1. 保证在我写之前所有的操作已经完成。2. 也只有在我完成此次"写"的操作的时候，下面的操作才能继续执行。
// 在AFNetworking中源码中，用于保护HTTPRequestHeaders的时候使用的是dispatch_barrier_sync(globalQueue, ^{ }); 
dispatch_barrier_async(globalQueue, ^{ });  
// 对于"读"的操作, 同步的获取值
dispatch_sync(globalQueue, ^{ });
```