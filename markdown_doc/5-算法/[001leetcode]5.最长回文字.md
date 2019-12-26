---
title: 001LeetCode 5. Longest Palindromic Substring 最长回文子串
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
- [研究三种解法](https://www.jianshu.com/p/631a5b2de027)：动态规划法、中心扩展法、马拉车(Manacher)算法
### 暴力解法
遍历所有子字符串，子串数为 N^2，长度平均为 N/2，时间复杂复O(N^3)
### 动态规划法
- [动态规划算法详解](https://www.cnblogs.com/hithongming/p/9229871.html)
- 思路：如果b[i][j]为true，那么b[i+1][j-1]也为true,b[i][j]表示从i到j的字符串是否为回文
- 复杂度：外层循环从后往前扫，内层循环从当前字符扫到结尾处，省略已经判断过的记录,时间复杂度O(N^2)
- 代码
```
Class Solution {
	 public String longestPalindrome(String s) {
        if (s.isEmpty()) return "";
		int n = s.length();
		//表示从i到j的字符串是否为回文
		boolean[][] b = new boolean[n][n];
		int maxlen = 0;
		String re = null;
		for(int i = n - 1; i >= 0; i--) {
			for(int j = i; j < n; j++){
				b[i][j] = s.charAt(i) == s.charAt(j) && (j - i + 1 < 3 || b[i+1][j-1]);
				if(b[i][j] && (re == null || maxlen < j - i + 1)) {
					maxlen = j - i + 1;
					re = s.substring(i, j + 1);
				}
			}
		}
		return re;
    }
}
```

### 中心扩展法
- 思路：假设i,或者i和i+1为回文中心，向两边扩展找出最大回文子串
- 复杂度: 分奇偶两种情况，以 i 为中心不断向两边扩展判断，无需额外空间，时间复杂度O(n^2)
- 代码：
```method1
Class Solution {
	 int start = 0, maxLen = 0;
    public String longestPalindrome(String s) {
        if(s.length() < 2) return s;
		
		for(int i = 0; i < s.length(); i++) {
			findPalindromicString(s, i, i);
			findPalindromicString(s, i, i + 1);
		}
		return s.substring(start, start + maxLen);
    }
    
    public void findPalindromicString(String s, int low , int high) {
		while(low >= 0 && high < s.length() && s.charAt(low) == s.charAt(high)) {
				low--;
				high++;
		}
		if(maxLen < high - low - 1) {
			start = low + 1;
			maxLen = high - low - 1;
		}
	}
}
```

### 马拉车(Manacher)算法
- 思路：
	- 中心扩展法的升级版本，无需考虑奇偶
	- 预处理将字符串空隙处添加特殊字符#，保证字符串一定是奇数