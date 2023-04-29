---
layout:       post
title:        "二叉搜索树的解题套路"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Algorithm
---

### 二叉搜索树是一种很特别、又很常用的一种的一种数据结构，同时它也很简单，因为它的定义不需要考虑平衡，所以解二叉搜索树的题一般会存在一个或者多个答案，一般只需要我们保证二叉搜索树的特性就可以了。

<!-- more -->

#### 1.中序遍历
> 为什么这里重点说一下中序遍历呢？由于二叉搜索树的特性所致，在对二叉搜索树中序遍历之后，会得到一个递增的数组，它的时间复杂度是O(n), 中序遍历很简单，我就说一下模板：

```swift
var result: [Int] = []
func inOrder(_ root: TreeNode?) {
    guard let root = root else { return }
    inOrder(root.left)
    result.append(root.val)
    inOrder(root.right)
}
```

#### 2.删除节点【包含找前继、后续节点、删除、替换、遍历功能】
> 其实个人认为在二叉搜索树中删除节点是最复杂的一个问题，基本上解决了这个问题之后，其他的问题都可以联系起来，并解决。

```swift
// 类型1：删除一个没有任何子节点的节点
func deleteNoChildNodeFromBST(_ root: TreeNode?) {
    // 这里是伪代码：1.1 首先获得当前节点的parent节点  1.2 而且还需要知道当前节点是parent节点的左节点还是右节点
    let info = getNodeInfo(root)
    info.direction == left ? info.parent?.left = nil : info.parent?.right = nil
}

// 类型2: 删除一个只有一个子节点的节点
func deleteOneChildNodeFromBST(_ root: TreeNode?) {
    let info = getNodeInfo(root)
    if info.left != nil {
        info.direction == left ? info.parent?.left = info.left : info.parent?.right = info.left
    } else {
        info.direction == left ? info.parent?.left = info.right : info.parent?.right = info.right
    }
}

// 类型3: 删除一个有两个节点的节点
func deleteTwoChildNodeFromBST(_ root: TreeNode?) {
    // 1.1 找到当前节点的前序或者后继节点
    let find = findProcessor(root)
    // 1.2 赋值给当前节点
    root.val = find.val
    // 1.3 删除find节点【这里的find节点肯定是只有一个子节点或者没有子节点的】
    (find.left == nil && find.right == nil) ? deleteNoChildNodeFromBST(find) : deleteOneChildNodeFromBST(find)
}

enum TreeNodeDirection {
    case left
    case right
}

// 这里可以使用中序、前序、后续、层序、递归等等办法，时间复杂度为O(n)遍历一遍就可以了
func getNodeInfo(_ root: TreeNode?) -> (parent: TreeNode?, direction: TreeNodeDirection) {
    //...
}

// 找前序节点【这里的操作和找后继节点是类似的】
func findProcessor(_ root: TreeNode?) -> TreeNode? {
    // 前序节点的特点是：当前节点左子树中最大的
    var leftChildNode = root.left

    while leftChildNode != nil {
        if (leftChildNode.right == nil) { // 找到了！！ }
        leftChildNode = leftChildNode.right
    }
}
```
#### 3. 说一道LeetCode题目450题
![leetcode450截图](/img/1592703704745.jpg)
> 解题如下：

```swift
//
//  LeetCode_450. 删除二叉搜索树中的节点_52.94% .swift
//  Coding
//
//  Created by GH on 2020/6/14.
//  Copyright © 2020 GH. All rights reserved.
//
import Foundation

/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     public var val: Int
 *     public var left: TreeNode?
 *     public var right: TreeNode?
 *     public init(_ val: Int) {
 *         self.val = val
 *         self.left = nil
 *         self.right = nil
 *     }
 * }
 */
class Solution {
    func deleteNode(_ root: TreeNode?, _ key: Int) -> TreeNode? {
        // 1. find it
        var find = findExactlyNode(root, nil, 0, key)
        // 2. delete it
        var root = root
        deleteExactly(&find, &root)

        return root
    }

    private func deleteExactly(_ root: inout (node: TreeNode?, parent: TreeNode?, direction: Int), _ before: inout TreeNode?) {
        guard let selfNode = root.node else { return }
        // has two children nodes
        if (selfNode.left != nil && selfNode.right != nil) {
            // 1. find processor / next
            let processor = findProcessor(selfNode)
            guard let processNode = processor.node else { return }
            // 2.  change value to process's value
            selfNode.val = processNode.val
            // 3. delete process node
            if processNode.left != nil && processNode.right != nil { return }

            var d = 1
            if let l = processor.parent?.left, let n = processor.node, l.val == n.val {
                d = -1
            }
            var processInfos = (node: processor.node, parent: processor.parent, direction: d)
            deleteExactly(&processInfos, &before)
        } else {
            guard let parentNode = root.parent else {
                // delete root node
                if selfNode.left != nil {
                    before = selfNode.left
                } else if(selfNode.right != nil) {
                    before = selfNode.right
                } else {
                    before = nil
                }
                return
            }
            if (selfNode.left == nil && selfNode.right == nil) {// has none of any child node
                if (root.direction == 1) {
                    parentNode.right = nil
                } else {
                    parentNode.left = nil
                }
            } else if (selfNode.left != nil) {
                if (root.direction == 1) {
                    parentNode.right = selfNode.left
                } else {
                    parentNode.left = selfNode.left
                }
            } else {
                if (root.direction == 1) {
                    parentNode.right = selfNode.right
                } else {
                    parentNode.left = selfNode.right
                }
            }
        }
    }
    /**
     * root: current node
     * parent:parent node
     * direction:
     *             Int: 1 = right
     *             Int: -1 = left
     *             Int: 0
     */
    private func findExactlyNode(_ root: TreeNode?, _ parent: TreeNode?, _ direction: Int, _ key: Int) -> (node: TreeNode?, parent: TreeNode?, direction: Int) {
        guard let root = root else { return (node: nil, parent: nil, direction: 0) }

        if (root.val == key) {
            return (node: root, parent: parent, direction: direction)
        } else if (root.val > key) {
            return findExactlyNode(root.left, root, -1, key)
        } else {
            return findExactlyNode(root.right,root, 1, key)
        }
    }

    private func findProcessor(_ root: TreeNode?) -> (node: TreeNode?, parent: TreeNode?) {
        guard let root = root else { return (node: nil, parent: nil) }

        var parentNode: TreeNode? = root
        var result: TreeNode? = root.left

        while result?.right != nil {
            parentNode = result
            result = result?.right
        }

        return (node: result, parent: parentNode)
    }
}
```