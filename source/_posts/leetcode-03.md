---
title: 3. Longest Substring Without Repeating Characters
date: 2021-06-14 11:22:49
category: LeetCode
tags: algorithm
---

Given a string `s`, find the length of the **longest substring** without repeating characters.

**Example 1:**

```
Input: s = "abcabcbb"
Output: 3
Explanation: The answer is "abc", with the length of 3.

```

<!-- more -->

**Example 2:**

```
Input: s = "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.

```

**Example 3:**

```
Input: s = "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3.
Notice that the answer must be a substring, "pwke" is a subsequence and not a substring.

```

**Example 4:**

```
Input: s = ""
Output: 0

```

**Constraints:**

- `0 <= s.length <= 5 * 104`
- `s` consists of English letters, digits, symbols and spaces.

这道题很容易想到可以用滑动窗口的办法解决，代码如下：

```cpp
// O(n^2)
int lengthOfLongestSubstring(string s) {
 
  if (s.size() <= 1) return s.size();

  int maxRange = 0;
  int left = 0, right;
  for (right = 1; right < s.size(); right++) {
    int equalIndex = -1;

    for (int i = right - 1; i >= left; i--) {
      if (s[right] == s[i]) {
        equalIndex = i;
        break;
      }
    }

    if (equalIndex != -1) {
      left = equalIndex + 1;
    }
    maxRange = max(maxRange, right - left + 1);

  }

  return maxRange;
}
```

对于 `0` 和 `1` 这种特殊情况，直接返回串的长度即可。

这里有一个优化可以省一点时间，那就是每次找到重复的字符时，事实上可以直接将 `left` 移动到 `equalIndex` 的下一位，之前的都可以不用管 (假设一步一步的移动，那么在下一次主循环中，马上又发现有重复的字符，且发生重复的位置不变，其实，最终的 `left` 一定是放在`equalIndex + 1`）。

这个解法虽然可以 `AC`，但每次查重复字符的时候还是有点耗时间的，那么能不能让查重快一点？

办法就是用**空间换时间：**

我们可以把 `equalIndex` 用一个数组记录下来，代码如下：

```cpp
int lengthOfLongestSubstring(string s) {
 
  if (s.size() <= 1) return s.size();

  vector<int> equalIndexes(128, -1);
  
  int maxRange = 0;
  int left = 0, right;
  for (right = 0; right < s.size(); right++) {
    int equalIndex = equalIndexes[s[right]];
	
    if (equalIndex != -1 && equalIndex >= left) {
      left = equalIndex + 1;
    }
    maxRange = max(maxRange, right - left + 1);
    equalIndexes[s[right]] = right;
  }

  return maxRange;
}
```
不过注意判断 equalIndex >= left，要保证不走回头路，因为下一个区间里的字符可能在之前的区间出现过。

这样一来的话，每个字符都只用访问一遍，时间复杂度优化到了 O(n)。