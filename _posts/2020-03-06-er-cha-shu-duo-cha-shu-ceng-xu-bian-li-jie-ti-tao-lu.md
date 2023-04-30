---
layout:       post
title:        "二叉树/多叉树 层序遍历解题套路"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Git
---

### 二叉树、多叉树个人认为是最考验一个递归思考算法的一种题型，今天就总结一下关于层序遍历的总结吧！

<!-- more -->


#### 先看一下二叉树的定义：

```swift
/**
 * Definition for a binary tree node.
  */
 public class TreeNode {
     public var val: Int
     public var left: TreeNode?
     public var right: TreeNode?
     public init(_ val: Int) {
         self.val = val
         self.left = nil
         self.right = nil
     }
 }

```

#### 再看一下多叉树的定义：

```swift
/**
 * Definition for a Node.
  */
 public class Node {
     public var val: Int
     public var children: [Node]
     public init(_ val: Int) {
         self.val = val
         self.children = []
     }
 }
```

#### 二叉树、多叉树套路模板

```swift
func displayTreeNode(_ root: TreeNode) {
    // 1: check data is right
    guard let root = root else { return }

    // 2: define a array
    var queue: [TreeNode] = [root]
    // 2.1: This value is tell you when will go to next level !!!!
    var levelSize: Int = 1

    while !queue.isEmpty {
        // 3. remve the first node
        let currentNode = queue.removeFirst()
        levelSize -= 1

        // 4. add currentNode's child nodes 
        if let l = currentNode.left { queue.append(l) }
        if let r = currentNode.right { queue.append(r) }
        // 4.1 
        let child = current.children
        if !child.isEmpty {
            queue = queue + child // queue = child + queue
        }

        // 5. when levelSize == 0 means that current level's nodes are all handled. will go to next level right now!!
        if (levelSize == 0)  { levelSize = queue.count }
    }
}

```

> 当然这个只是一种套路，但是还有递归等其他的操作。后续慢慢补上吧！

