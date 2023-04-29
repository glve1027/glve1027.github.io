---
layout:       post
title:        "关于链表LinkedList解题心得"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Algorithm
---

### 关于链表，近期在刷题的时候发现，它还是有操作的套路可言的，不见得所有的题目都能套上模板，但是已经能解决60-70%的链表题型了。套路模板如下：

<!-- more -->

1. 入参给你个ListNode, 也可以有其他参数，根据条件返回他指定类型的链表。
2. 需要临界值的判断。
3. （可以建立一个dummp节点），为了简化操作
4. 遍历循环 【4.1: 先报值保留下来 4.2:迭代】
5. 返回dummp.next节点

```java
public ListNode xx(ListNode head, int val) {
        if (head == null) { return head; } // 如第二点
        // 这里是个假数据，简化判断用的 - 如第三点
        ListNode dummy = new ListNode(-1);
        ListNode node = dummy;
        while (head != null) {
            // 先把值保留下来 - 如4.1
            ListNode temp = head;
            // 在迭代 - 如4.2
            head = head.next;
            // 也可以根据条件提前返回

            if (temp.val != val) {
                node.next = temp;
                temp.next = null;

                node = node.next;
            }
        }
        return dummy.next; // 如5
    }

class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) { return head; }
        // 这里是个假数据，简化判断用的
        ListNode dummy = new ListNode(-1);
        ListNode node = dummy;
        int lastValue = -1;
        boolean first = true;
        while (head != null) {
            // 先把值保留下来
            ListNode temp = head;
            // 在迭代
            head = head.next;

            if (first || lastValue != temp.val) {
                node.next = temp;
                temp.next = null;
                if (first) { first = false; }

                lastValue = temp.val;
                node = node.next;
            }
        }
        return dummy.next;
    }
}
```

![leetCode203截图](/img/1582629779038.jpg)
```java
/**
     * LeetCode 203
     * @param head
     * @param val
     * @return
     */
    public ListNode removeElements(ListNode head, int val) {
        if (head == null) { return head; }
        // 这里是个假数据，简化判断用的
        ListNode dummy = new ListNode(-1);
        ListNode node = dummy;
        while (head != null) {
            // 先把值保留下来
            ListNode temp = head;
            // 在迭代
            head = head.next;

            if (temp.val != val) {
                node.next = temp;
                temp.next = null;

                node = node.next;
            }
        }
        return dummy.next;
    }
```

![leetCode206截图](/img/1582629898087.jpg)
```java
/**
     * LeetCode 206 翻转链表 - 利用之前节点的值重新创建节点拼接
     * @param head
     * @return
     */
    public ListNode reverseList(ListNode head) {
        if (head == null) { return null; }
        ListNode resultNode = null;
        while (head != null) {
            if (resultNode == null) {
                resultNode = new ListNode(head.val);
            } else {
                ListNode temp = new ListNode(head.val);
                temp.next = resultNode;

                resultNode = temp;
            }
            head = head.next;
        }
        return resultNode;
    }

    /**
     * LeetCode 206 翻转链表 - 不重新创建节点
     * @param head
     * @return
     */
    public ListNode reverseList_01(ListNode head) {
        if (head == null || head.next == null) return head;

        ListNode resultNode = null;
        while (head != null) {
            ListNode temp = head;

            head = head.next;

            if (resultNode == null) {
                temp.next = null;
                resultNode = temp;
            } else {
                temp.next = resultNode;
                resultNode = temp;
            }
        }
        return resultNode;
    }

    /**
     * LeetCode 206
     * @param head
     * @return
     */
    public ListNode reverseList_02(ListNode head) {
        if (head == null || head.next == null) { return head; }

        ListNode newHead = null;
        while (head != null) {
            ListNode temp = head.next;

            head.next = newHead;
            newHead = head;

            head = temp;
        }
        return newHead;
    }
    /**
     * LeetCode 206 递归的解法
     * @param head
     * @return
     */
    public ListNode reverseList_03(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode newNode = reverseList_03(head.next);
        head.next.next = head;
        head.next = null;
        return newNode;
    }
```

![leetCode237截图](/img/1582629978222.jpg)
```java
/**
     * LeetCode 237 删除一个节点
     * @param node
     */
    public void _237_deleteNode(ListNode node) {
        if (node.next == null) {
            node = null;
        } else {
            node.val = node.next.val;
            node.next = node.next.next;
        }
    }

class Solution {
    public ListNode middleNode(ListNode head) {
        if (head == null || head.next == null) { return head; }
        ListNode slow = head;
        ListNode fast = head.next;
        while (fast != null) {
            if (fast.next == null) { return slow.next; }
            fast = fast.next.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

![leetCode141截图](/img/1582629688771.jpg)
```java
 /**
     * LeetCode 141 判断链表是否存在环
     * @param head
     * @return
     */
    public boolean _141_hasCycle(ListNode head) {
        if (null == head || null == head.next) { return false; }
        ListNode slow = head;
        ListNode fast = head.next;
        while (slow != null && fast != null) {
            if (slow.equals(fast)) return true;
            if (fast.next == null) { return false; }
            fast = fast.next.next;
            slow = slow.next;
        }
        return false;
    }

    /**
     * LeetCode 141 配合HashMap解题
     * @param head
     * @return
     */
    public boolean _141_hasCycle_01(ListNode head) {
        if (null == head || null == head.next) { return false; }
        HashMap<ListNode, Integer> map = new HashMap<>();

        while (head != null) {
            if (map.containsKey(head)) { return true; }
            map.put(head, 1);
            head = head.next;
        }

        return false;
    }
```