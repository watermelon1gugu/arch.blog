---
title: leetcode-6 题解
date: 2018-08-12 17:07:20
tags: leetcode
---

# 题目

将字符串 `"PAYPALISHIRING"` 以Z字形排列成给定的行数：

```
P   A   H   N
A P L S I I G
Y   I   R
```

之后从左往右，逐行读取字符：`"PAHNAPLSIIGYIR"`

实现一个将字符串进行指定行数变换的函数:

```
string convert(string s, int numRows);
```

<!--*more*-->

**示例 1:**

```
输入: s = "PAYPALISHIRING", numRows = 3
输出: "PAHNAPLSIIGYIR"
```

**示例 2:**

```
输入: s = "PAYPALISHIRING", numRows = 4
输出: "PINALSIGYAHRPI"
解释:

P     I    N
A   L S  I G
Y A   H R
P     I
```

# 直接上代码#

```c++
using namespace std;
class Solution {
public:
    string convert(string s, int numRows) {
        if(numRows==1){
            return s;
        }
        int step[2];
        string result;
        for(int i = 0;i<numRows;i++){
            step[result.length()%2] = (numRows-1-i)*2==0?2*i:(numRows-1-i)*2;
            step[!(result.length()%2)] = 2*i==0?(numRows-1-i)*2:2*i;
            for(int col = i;col<s.length();col+= step[!(result.length()%2)]){
                result.push_back(s[col]);
            }
        }
        return result;
    }
};
```

为了省去step的选择标记，使用result字符串长度作为标记。