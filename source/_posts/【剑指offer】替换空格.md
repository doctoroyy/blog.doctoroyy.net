---
title: 【剑指offer】替换空格
date: 2020-01-07 23:09:39
tags: 刷题
---


## 替换空格

#### 题目描述：

> 请实现一个函数，将一个字符串中的每个空格替换成“%20”。
例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

<!-- more -->

#### 分析：


> 首先考虑从前往后替换空格的情况：每移动一个空格就需要把后面的所有字符移动三位
，这样做的时间复杂度是O(n^2)

> 能不能换个思路，从后往前替换？

> 我们可以事先计算出所有空格替换后的字符串长度，每替换一个字符串长度+2，
因此目标字符串长度为 numOfSpace * 2 + |str| 
(其中numOfSpace 代表空格个数，|str|代表原字符串长度)

> 维护两个指针p1, p2，p1指向原字符串末尾，p2指向目标字符串的末尾；
 接下来从后往前遍历整个字符串，遇到普通字符，直接将p1位置的字符移动到当前p2指向的位置，
然后p1, p2向前移动一位；
遇到空格，则将空格替换为"%20", p2移动三位，p1移动一位

> 需要注意的是p1不能超越边界即保持p1不等于p2，
例如出现" abc"这样的情况，当p1,p2都指向0时，就不能再往前移动了



#### 代码实现：

```C++
class Solution {
public:
  void replaceSpace(char *str, int len) {
    int numOfSpace = 0;
    for (int i = 0; i < len; i++) {
      if (str[i] == ' ') numOfSpace++;
    }
    int newStrLen = len + numOfSpace * 2;
    int p1 = len, p2 = newStrLen;//字符串的末尾是'\0'，所以p1指向len而不是len - 1
    while (p1 != p2) {
      while (str[p1] != ' ') {
        str[p2--] = str[p1--];
      }
      str[p2--] = '0';
      str[p2--] = '2';
      str[p2--] = '%';
      p1--;
    }
  }
};
```