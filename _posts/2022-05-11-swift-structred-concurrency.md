---
layout:       post
title:        "Summary of structred concurrency"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Swift
---

### How to use structred concurrency?
* Only use these two keywords: `async`, `await`. You can make it.
* For Example:

```swift
// 1. normal function
func hello(_ input: Int) -> String
// 2. call the normal function
let returnValue = hello(123)

// 1.1 structred concurrency function
func hello(_ input: Int) async -> String
// 1.2 call the above function
// but will cause build error. Error message: 'async' call in a function that does not support concurrency.
let returnValue = await hello(123)

// 1.2.1 How to call this async method correctly
Task {
    let returnValue = await hello(123)
}
```

### What doest `Task` do?

> Task: A unit of asynchronous work.
> When you create an instance of `Task`,
> you provide a closure that contains the work for that task to perform.
> Tasks can start running immediately after creation;
> you don't explicitly start or schedule them.
> After creating a task, you use the instance to interact with it ---
> A task's execution can be seen as a series of periods where the task ran.

* Above informations are come from [Apple developer doc](https://developer.apple.com/documentation/swift/task)
* Based on this `Task` keyword. We can rewrite above function like this:

```swift
func hello(_ input: Int) async -> String {
    let task = Task { () -> String in 
        // async function to fetch this response
    }

    let response = await task.value
    //...
}
```

### Why don't we need execute async method on the main Queue.

* Because `await` will block current execution, and will not return any response unitl finished executing `async` method.

### Is `structred concurrenty` really efficient??

* Yes or Not.

> Answer: Yes. Apple apple guarantees that.
> But In some cases we need to be careful.

* For Example:

```swift
func hello(_ input: Int) async -> String
func hello1(_ input: Int) async -> String

let result1 = await hello(123)
let result2 = await hello1(123)
// If the top two functions are not related. This is inefficient.
```

### We can use Async let to avoid inefficient.

```swift
async let r1 = hello(123)
async let r2 = hello1(123)
let (result1, result2) = await (r1, r2)
```


