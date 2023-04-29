---
layout:       post
title:        "DFS的思考"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Algorithm
---

> 二叉树和多叉树的问题天生就会和`分治`和`递归`扯上千丝万缕的关系，那么DFS这类题应该具体怎么考虑呢？这里总结三种题型。

<!-- more -->

* 左右两边完全可以自己处理自己的数据，返回的数据也是`单一`的。
* 怎么理解这里的`单一`呢？
> 就是只需要返回一个值就能解决问题，这里举个模板：

```swift
func dfs(_ root: TreeNode?) {
    guard let root = root else {
        return xxx
    }
    // 计算左边的结果
    left_results = self.dfs(root.left)

    // 计算右边的结果
    right_results = self.dfs(root.right)

    // 计算当前root的结果
    return finalHandle(root + left_results + right_results)
}
```

* 也有可能每个节点需要携带一些信息：例如Leecode的这一题：【https://leetcode.com/submissions/detail/784462690/】
* 需要每一个节点都带上最长的链，最长直径的信息

```swift
class Solution {
    func diameterOfBinaryTree(_ root: TreeNode?) -> Int {
        return self.dfs(root).max_diameter
    }
    
    private func dfs(_ root: TreeNode?) -> (max_diameter: Int, max_chain: Int) {
        guard let root = root else {
            return (max_diameter: 0, max_chain: 0)
        }
        let (left_max_diameter, left_max_chain) = self.dfs(root.left)
        let (right_max_diameter, right_max_chain) = self.dfs(root.right)
        
        let max_diameter = max(left_max_diameter, right_max_diameter, left_max_chain + right_max_chain)
        let max_chain = max(left_max_chain, right_max_chain) + 1
        return (max_diameter: max_diameter, max_chain: max_chain)
    }
}
```

* 最后一种：可能需要的是一个全局的变量。例如Leetcode[https://leetcode.com/problems/validate-binary-search-tree/]
* 对于这种应该在参数中携带上，类似`DFS`中常常用的`visited`的这一个结合一样。

```swift
class Solution {
    func isValidBST(_ root: TreeNode?) -> Bool {
        return self.dfs(root, Int.min, Int.max)
    }
    
    func dfs(_ root: TreeNode?, _ min: Int, _ max: Int) -> Bool {
        guard let root = root else {
            return true
        }
        
        if root.val >= max || root.val <= min {
            return false
        }
        
        return self.dfs(root.left, min, root.val) && self.dfs(root.right, root.val, max)
    }
}
```

* 当然如果你选择在每个节点上都带上最大，最小值也是可以的，但是效率不高。

```swift
class Solution {
    func isValidBST(_ root: TreeNode?) -> Bool {
        return self.dfs(root).valid
    }
    
    func dfs(_ root: TreeNode?) -> (valid: Bool, min: Int?, max: Int?) {
        guard let root = root else {
            return (valid: true, min: nil, max: nil)
        }
        let (left_valid, left_min, left_max) = self.dfs(root.left)
        let (right_valid, right_min, right_max) = self.dfs(root.right)
        
        if !left_valid || !right_valid {
            return (valid: false, min: nil, max: nil)
        }
     
        if let left_value = root.left, root.val <= left_value.val {
            return (valid: false, min: nil, max: nil)
        }
        
        if let right_value = root.right, root.val >= right_value.val {
            return (valid: false, min: nil, max: nil)
        }
        
        if let left_max = left_max, left_max >= root.val {
            return (valid: false, min: nil, max: nil)
        }
        
        if let right_min = right_min, right_min <= root.val {
            return (valid: false, min: nil, max: nil)
        }
        
        return (valid: true, min: left_min ?? root.val, max: right_max ?? root.val)
    }
}
```