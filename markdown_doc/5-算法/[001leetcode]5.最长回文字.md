---
title: [001LeetCode] 5. Longest Palindromic Substring 最长回文子串
tags: LeetCode-0
renderNumberedHeading: true
grammar_cjkRuby: true
---
## [001-LeetCode5] 最长回文子串
example1
```
Input: "babad"
Output: "bab"
```
example2
```
Input: "cbbd"
Output: "bb"
```
- 回文字就是正着读和反着读都一样的字符串，分为两种奇数回文和偶数回文。
- 研究三种解法：动态规划法、中心扩展法、马拉车(Manacher)算法
### 暴力解法
遍历所有子字符串，子串数为 N^2，长度平均为 N/2，时间复杂复O(N^3)
### 动态规划法
- [动态规划算法详解](https://www.cnblogs.com/hithongming/p/9229871.html)
- 本题思路：最大回文子串的

### 中心扩展法

### 马拉车(Manacher)算法