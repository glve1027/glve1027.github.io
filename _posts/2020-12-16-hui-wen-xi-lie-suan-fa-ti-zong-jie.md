---
layout:       post
title:        "回文系列算法题总结"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Algorithm
---

### 在我刚开始工作的时候，很多公司都喜欢考回文的算法，知道现在依旧很多大小厂都喜欢考这类题目，这里总结一下解法

<!-- more -->

#### 1. 动态规划解题 --> 时间复杂度：(O(N^2))
> 动态规划的问题重点就是：理清楚状态转移方程
> 计算最长回文子串算法题：

```java
public String longestPalindrome(String s) {
    if (s == null || s.equals("")) {
        return "";
    }


    int n = s.length();
    boolean[][] isPalindrome = new boolean[n][n];

    // 初始化
    // 1个的
    int start = 0;
    int longest = 1;
    for (int i = 0; i < n; i++) {
        isPalindrome[i][i] = true;
    }

    // 2个的
    for (int i = 0; i < n - 1; i++) {
        isPalindrome[i][i+1] = s.charAt(i) == s.charAt(i+1);
        if (isPalindrome[i][i+1]) {
            start = i;
            longest = 2;
        }
    }

    // x  abba   y
    // ^         ^
    // i         j
    // 状态转移方程 isPalindrome[i][j] = isPalindrome[i+1][j-1] && s.charAt(i) == s.charAt(j)
    for (int i = n - 1; i >= 0; i--) {
        for (int j = i + 2; j < n; j++) {
            isPalindrome[i][j] = isPalindrome[i+1][j-1] && s.charAt(i) == s.charAt(j);
            if (isPalindrome[i][j] && j - i + 1 > longest) {
                start = i;
                longest = j - i + 1;
            }
        }
    }

    return s.substring(start, start + longest);
}
```

* 计算最长回文子序列的长度算法题： 子串和子序列还是有本质上差别的这里要理清楚

```java
/**
* @param s: the maximum length of s is 1000
* @return: the longest palindromic subsequence's length
*/
public int longestPalindromeSubseq(String s) {
// write your code here
if (s == null || s.equals("")) {
    return 0;
}

// 字符长度
int n = s.length();
// 定义一个二维数组：longestPalindrome[i][j] 表示 坐标i与坐标j之间的最长子序列的值
int[][] longestPalindrome = new int[n][n];
// 初始化
// 1. 赋值1个字符的情况
int longest = 1;
for (int i = 0; i < n; i++) {
    longestPalindrome[i][i] = 1;
}

// 2. 赋值2个字符的情况
for (int i = 0; i < n - 1; i++) {
    longestPalindrome[i][i+1] = s.charAt(i) == s.charAt(i+1) ? 2 : 1;
    if (longestPalindrome[i][i+1] == 2) {
        longest = 2;
    }
}

// 3. 其他情况
// 状态转移方程
// longestPalindrome[i][j] = s.charAt(i) == s.charAt(j) ?
//                           longestPalindrome[i + 1][j - 1] + 2
//                           max(longestPalindrome[i][j - 1], longestPalindrome[i + 1][j])
for (int i = n; i >= 0; i--) {
    for (int j = i + 2; j < n; j ++) {
        longestPalindrome[i][j] = s.charAt(i) == s.charAt(j) ? longestPalindrome[i + 1][j - 1] + 2 : Math.max(longestPalindrome[i][j - 1], longestPalindrome[i + 1][j]);
        if (longestPalindrome[i][j] > longest) {
            longest = longestPalindrome[i][j];
        }
    }
}

return longest;
}
```

#### 2. 中心线枚举法 --> 时间复杂度：(O(N^2))
> 计算最长回文子串算法题

```java
public String longestPalindrome(String s) {
    if (s == null || s.equals("")) {
        return "";
    }

    String longestString = "";
    for (int i = 0; i < s.length(); i++) {
        // odd
        String odd_longestPalindrome = getSubString(s, i, i);
        if (odd_longestPalindrome.length() > longestString.length()) {
            longestString = odd_longestPalindrome;
        }

        // even
        String even_longestPalindrome = getSubString(s, i, i + 1);
        if (even_longestPalindrome.length() > longestString.length()) {
            longestString = even_longestPalindrome;
        }
    }
    return longestString;
}

private String getSubString(String s, int left, int right) {
    while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
        left--;
        right++;
    }
    return s.substring(left + 1, right);
}
```

#### 3. 暴力解法【我这里就不写了，你可以自己尝试一下】


