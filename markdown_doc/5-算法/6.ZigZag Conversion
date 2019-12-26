---
title: 6.ZigZag Conversion
tags: LeetCode-0
renderNumberedHeading: true
grammar_cjkRuby: true
---
example1
```
Input: s = "PAYPALISHIRING", numRows = 3
Output: "PAHNAPLSIIGYIR"
P   A   H   N
A P L S I I G
Y   I   R
```
example2
```
Input: s = "PAYPALISHIRING", numRows = 4
Output: "PINALSIGYAHRPI"
Explanation:
P     I    N
A   L S  I G
Y A   H R
P     I
```

思路：
对a-z的字符串按numRows = 5做ZigZag转换，索引位置如下
![a-z](./images/1577362013806.png)
规律:
- 第一行和最后一行，每个字符之间的index相差（numRows-1)*2=8
- 第二行：1+6=7，7+2=9，9+5=15，15+2=17...差值6和2交替，6+2=8，2=2*1（1为数组第二行索引，后面同理）；
- 第三行：2+4=6，6+4=10，10+4=14...差值4+4=8，4=2*2；
- 第四行：3+2=5， 5+6=11，11+2=13...差值2+6=8，6=2*3；
- 第二、三、四行，索引为i；对应的规律是每行相邻字符，索引相差（numRows-1)*2 - i*2、i*2交替
代码：
```
class Solution {
    public String convert(String s, int numRows) {
		//注意边界情况
		if (numRows <= 1) return s;
    	char[] newStrChars = new char[s.length()];
		// 新字符串索引
		int curIndex = 0;
		// 遍历每一行
		for(int i = 0; i < numRows; i++) {
			// 计算每一行索引应该的移动的值
			int[] moves = {(numRows-1) * 2 - i*2, i*2};
			if(i == 0 || i == (numRows - 1)) {
				moves[0] = (numRows-1) * 2;
				moves[1] = (numRows-1) * 2;
			}
			// 使用tempIndex移动旧字符串的位置
			int tempIndex = i;
			int moveIndex = 0;
			while(tempIndex < s.length()) {
				newStrChars[curIndex++] = s.charAt(tempIndex);
				tempIndex += moves[(moveIndex++) % 2];
			}
		}
		return new String(newStrChars);
    }
}
```