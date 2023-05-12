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

### More complicated situation:
* We can use `withTaskGroup` or `withThrowingTaskGroup`. It you want to use above ones. You must return the same type. such as `enum`

```swift
enum ReturnType {
    case returnVoid
    case returnIntArray([Int])
    case returnString(String)
}
    
func hello() async -> [ReturnType] { return [.returnVoid] }
    
func hello1() async -> [ReturnType] { return [.returnIntArray([1,2,3])] }
    
func hello2() async -> [ReturnType] { return [.returnString("hello")] }

func test() async {
    let complexResult = await withTaskGroup(of: [ReturnType].self, body: { group -> [ReturnType]  in
        group.addTask {
            // Specific operation
            await self.hello()
        }
            
        group.addTask {
            // Specific operation1
            await self.hello1()
        }
            
        group.addTask {
            await self.hello2()
        }
            
        let finalResult: [ReturnType] = await group.reduce([], +)
            
        return finalResult
    })
    print(complexResult)
}
```
### Easy to test:
> If these functions are all private. Only one public function: func test() async. Which means we just need to test this function.
* Before we start wirting the test codes. we need to update some codes first.

1. Add `private` for these three functions: `hello`, `hello1`, `hello2`.
2. Update above thress functions to `static` functions or we can move to another object, because we use these three functions as the default parameters.
3. Add these three parameters for this function `test`
4. Add `return type` to easy test.

```swift
  private static func hello() async -> [ReturnType] { return [.returnVoid] }
  
  private static func hello1() async -> [ReturnType] { return [.returnIntArray([1,2,3])] }
  
  private static func hello2() async -> [ReturnType] { return [.returnString("hello")] }
  
  func test(_ func1: @escaping () async -> [ReturnType] = TestAsync.hello,
            _ func2: @escaping () async -> [ReturnType] = TestAsync.hello1,
            _ func3: @escaping () async -> [ReturnType] = TestAsync.hello2) async -> [ReturnType] {
    let complexResult = await withTaskGroup(of: [ReturnType].self, body: { group -> [ReturnType]  in
      group.addTask {
        // Specific operation
        await func1()
      }

      group.addTask {
        // Specific operation1
        await func2()
      }

      group.addTask {
        await func3()
      }

      let finalResult: [ReturnType] = await group.reduce([], +)

      return finalResult
    })
    return complexResult
  }
```
* Test Codes:
```swift
  func testFinalTestFunc() async {
    
    let result = await TestAsync().test {
      return [.returnVoid]
    } func2: {
      return [.returnIntArray([1])]
    } func3: {
      return [.returnString("hello")]
    }
    
    XCTAssertEqual(result[0], .returnVoid)
    XCTAssertEqual(result[1], .returnIntArray([1]))
    XCTAssertEqual(result[2], .returnString("hello"))
  }
```


