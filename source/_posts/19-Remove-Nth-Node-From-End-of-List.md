---
title: 19. Remove Nth Node From End of List
date: 2019-07-22 23:49:21
categories: LeetCode
tags: 刷题
---

Given a linked list, remove the n-th node from the end of list and return its head.

**Example:**
```
Given linked list: 1->2->3->4->5, and n = 2.

After removing the second node from the end, the linked list becomes 1->2->3->5.
```
<!-- more -->
**Note:**

Given n will always be valid.

来自LeetCode上的一道链表操作题，题目意思是删除链表的倒数第n个元素，由于数据保证n是合法的，所以只需要考虑两个边界情况：1.删除的是首节点 2.删除的是尾节点（其实这个情况不用考虑2019.8.11）

接下来附上AC代码：

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
  public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
      int len = 0;
      for (auto it = head; it != NULL; it = it->next) len++;
      // cout << len << endl;
      //[1,2,3,4,5] 2  
      int moveSteps = len - n - 1;
      if (moveSteps == -1) { //说明要删除的是头结点 
        return head->next;
      }
      ListNode* p = head;
      for (int i = 0; i < moveSteps; i++) {
        p = p->next;
      }
      //if (moveSteps == len - 2) {//说明删除的是尾节点 
      //  p->next = NULL;
      //} else {
        p ->next = p->next->next;
      //}
      return head;
    }
};
```
