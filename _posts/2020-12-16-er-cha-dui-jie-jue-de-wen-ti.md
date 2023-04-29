---
layout:       post
title:        "二叉堆解决的问题"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Algorithm
---

### 二叉堆能解决什么问题？ 还是要有几个先决条件的。
1. 需要数据之间是可以比较的。
2. 解决类似找到第K个问题的。

<!-- more -->

* 我也写了一个[Swift版本的二叉堆](https://github.com/glve1027/BinaryHeap), 有兴趣的可以查看一下源码, 利用二叉堆，我们也可以实现优先队列（Priority Queue）其实JAVA中的队列就是用二叉堆来实现的。
* 二叉堆的两个问题，需要注意。

1. 如何生成二叉堆，是有一定规律的，这个规律就是：如果是大顶堆，那就是父 > 子 | 小顶堆则 反过来。
2. 这里我将这个过程叫做Heapify，这个规律就是1. 从上而下上滤 2.从下而上下滤 [两个在时间复杂度上有细微的差别，可以参考下面的代码，推荐2]

```swift
  /**
     * 1. 自上而下上滤
     * 2. 自下而上下滤
     */
    private func heapify() {
        /* for i in 0..<elements.count { shiftUp(index: i) } */
        let firstNoChildIndex = (self.elements.count >> 1) - 1
        for i in (0...firstNoChildIndex).reversed() { shifDown(index: i) }
    }
```

> 至于上面提到的如何上滤/下滤就是另一个重点，需要不断的比较、覆盖值、修改索引，代码如下: 这里有个关键点，就是不要每次在循环的时候，就完成父与子之间的值交换，只需要记住就行了

```swift
/**
     * 上滤
     * 参数为index索引
     */
    private func shiftUp(index: Int) {
        guard index < self.elements.count && index >= 0 else { return }
        let currentValue: E = self.elements[index];
        var index = index
        
        while index > 0 {
            // 找到parentIndex
            let parentIndex = (index - 1) >> 1;
            if (parentIndex < 0) { break }
            
            let parentValue = self.elements[parentIndex]
            // 如果子 <= parent的话，就直接退出
            if (currentValue <= parentValue) { break }
            
            // 否则就替换值
            self.elements[index] = parentValue
            // 并且替换索引
            index = parentIndex
        }
        
        self.elements[index] = currentValue
    }
    
    /**
     * 下滤
     * 参数为index索引
     */
    private func shifDown(index: Int) {
        guard index < self.elements.count && index >= 0 else { return }
        let currentValue = self.elements[index]
        var index = index
        
        // 找到第一个没有子节点的节点坐标
        let half = self.elements.count >> 1
        
        while (index < half) {
            // 找到子节点
            // 两种情况
            // 1. 只有左子节点
            // 2. 有左右两个子节点
            var childIndex = (index << 1) + 1
            guard childIndex < self.elements.count else { break }
            
            var childValue = self.elements[childIndex]
            // 找出子节点中较大的那个
            if (childIndex + 1 <= self.elements.count - 1  && self.elements[childIndex + 1] > childValue) {
                childValue = self.elements[childIndex + 1]
                childIndex += 1
            }
            
            // 判断大小
            if (currentValue >= childValue) { break }
            
            // 替换
            self.elements[index] = childValue
            // 修改索引
            index = childIndex
        }
        self.elements[index] = currentValue
    }
```

> 参考一道LeeCode
![leetcode 215截图](/img/1597052570350.jpg)
> 解题如下

```swift
class Solution {
    func findKthLargest(_ nums: [Int], _ k: Int) -> Int {
        let binary = BinaryHeap(e: nums)
        var index = 1
        guard index <= k else { return -1 }
        while index < k {
            _ = binary.remove()
            index += 1
        }
        return binary.remove() ?? -1
    }
}

class BinaryHeap<E: Comparable> {
    public var elements: [E] = []
    // Heapify
    init(e: [E]) {
        guard !e.isEmpty else { return }
        e.forEach { elements.append($0) }
        guard elements.count > 1 else { return }
        heapify()
    }
    
    // add
    public func add(element: E) {
        // 1. append element
        elements.append(element)
        // 2. shift up
        shiftUp(index: elements.count - 1)
    }
    
    /**
     * 1. 自上而下上滤
     * 2. 自下而上下滤
     */
    private func heapify() {
        /* for i in 0..<elements.count { shiftUp(index: i) } */
        let firstNoChildIndex = (self.elements.count >> 1) - 1
        for i in (0...firstNoChildIndex).reversed() { shifDown(index: i) }
    }
    
    /**
     * 上滤
     * 参数为index索引
     */
    private func shiftUp(index: Int) {
        guard index < self.elements.count && index >= 0 else { return }
        let currentValue: E = self.elements[index];
        var index = index
        
        while index > 0 {
            // 找到parentIndex
            let parentIndex = (index - 1) >> 1;
            if (parentIndex < 0) { break }
            
            let parentValue = self.elements[parentIndex]
            // 如果子 <= parent的话，就直接退出
            if (currentValue <= parentValue) { break }
            
            // 否则就替换值
            self.elements[index] = parentValue
            // 并且替换索引
            index = parentIndex
        }
        
        self.elements[index] = currentValue
    }
    
    /**
     * 下滤
     * 参数为index索引
     */
    private func shifDown(index: Int) {
        guard index < self.elements.count && index >= 0 else { return }
        let currentValue = self.elements[index]
        var index = index
        
        // 找到第一个没有子节点的节点坐标
        let half = self.elements.count >> 1
        
        while (index < half) {
            // 找到子节点
            // 两种情况
            // 1. 只有左子节点
            // 2. 有左右两个子节点
            var childIndex = (index << 1) + 1
            guard childIndex < self.elements.count else { break }
            
            var childValue = self.elements[childIndex]
            // 找出子节点中较大的那个
            if (childIndex + 1 <= self.elements.count - 1  && self.elements[childIndex + 1] > childValue) {
                childValue = self.elements[childIndex + 1]
                childIndex += 1
            }
            
            // 判断大小
            if (currentValue >= childValue) { break }
            
            // 替换
            self.elements[index] = childValue
            // 修改索引
            index = childIndex
        }
        self.elements[index] = currentValue
    }
    
    // remove
    public func remove() -> E? {
        guard !self.elements.isEmpty else { return nil }
        let firstElement = self.elements.removeFirst()
        guard !self.elements.isEmpty else { return firstElement }
        let lastElement = self.elements.removeLast()
        self.elements.insert(lastElement, at: 0)
        // 下滤第一个
        shifDown(index: 0)
        return firstElement
    }
    
    // clear
    public func clear() {
        elements.removeAll()
    }
}
```

