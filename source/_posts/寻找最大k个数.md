---
title: '寻找最大k个数'
date: 2019-12-23 00:03:22
categories: 算法
tags:
  - 二叉堆
  - 排序
---

## 如何在一个无序的数组中寻找最大的k个数？


对于这个问题， 最容易想到的办法就是给数组排个序, 如果使用快速排序，时间复杂度是O(nlogn)， 但事实上我们只需要寻找这k个数，排序的方法显然做了多余的事情。

那么，能不能在这个基础上优化一下呢？

<!-- more -->

## 基于快速排序的优化 O(klogn)

我们知道，快速排序的思想是：在数组中选中一个支点，以这个支点将数组划分成两部分s1, s2，保证s1所有元素小于支点，s2不小于（用“不小于”好像严谨一些）支点。
现在，我们修改一下，让s2中所有元素不小于支点，s2中所有元素小于支点。假设支点所在位置为i，那么：
- 当 i < k 时，我们已经找到了最大k个数中的i个，只需要在右边部分寻找剩下的 k - i 个数；
- 当 i > k 时，那么在前 i - 1 个数当中寻找最大的k个数；
- 当 i == k 时，前 i 个数就是我们要找的最大k个数；

接下来给出代码实现：
```JavaScript
function swap(arr, i, j) {
  [arr[i], arr[j]] = [arr[j], arr[i]];
}

function partition(arr, low, high) {
  let p = arr[low], m = low;
  for (let i = low + 1; i <= high; i++) {
    if (arr[i] > p) {
      m++;
      swap(arr, i, m);
    }
  }
  swap(arr, low, m);
  return m;
}

function quick_sort(arr, low, high) {
  if (low < high) {
    let m = partition(arr, low, high);
    quick_sort(arr, low, m - 1);
    quick_sort(arr, m + 1, high);
  }
}

function find_top_k(arr, low, high, k) {
  if (low < high) {
    let m = partition(arr, low, high);
    let cnt = m - low + 1;
    if (cnt > k) {
      find_top_k(arr, low, m - 1, k);
    } else if (cnt < k) {
      find_top_k(arr, m + 1, high, k - cnt);
    }
  }
}
```

## 基于堆排序的优化 O(klogn)

首先需要了解一下堆（二叉堆）的概念：
 > 堆（英语：Heap）是计算机科学中的一种特别的树状数据结构。若是满足以下特性，即可称为堆：“给定堆中任意节点P和C，若P是C的母节点，那么P的值会小于等于（或大于等于）C的值”。若母节点的值恒小于等于子节点的值，此堆称为最小堆（min heap）；反之，若母节点的值恒大于等于子节点的值，此堆称为最大堆（max heap）。在堆中最顶端的那一个节点，称作根节点（root node），根节点本身没有母节点（parent node）。
 
 构建一个最大堆：
 
 ```JavaScript
 function heapify(arr, start, end) {
  let parent = start;
  let child = parent * 2 + 1;
  // 选取两个子节点中较大的（如果有两个的话）
  child = (arr[child + 1] > arr[child] && child + 1 <= end) ? child + 1 : child; 
  if (arr[child] > arr[parent] && child <= end) {
    swap(arr, child, parent);
    // 继续向下探寻，如果找到子元素大于当前根节点，继续交换，一直到数组尾部为止
    heapify(arr, child, end); 
  }
}

 for (let i = parseInt(len / 2) - 1; i >= 0; i--) { // 从每个根节点开始往下寻找大于根节点的元素
   heapify(arr, i, len - 1);
 }
 ```
 
  堆排序：

 ```JavaScript
function heap_sort(arr, len) {
  for (let i = parseInt(len / 2) - 1; i >= 0; i--) {
    heapify(arr, i, len - 1);
  }
  for (let i = len - 1; i > 0; i--) {
    swap(arr, 0, i); //把堆顶放入数组尾部
    heapify(arr, 0, i - 1); // 接下来在区间[0, i - 1]之间继续构造最大堆
  }
}
 ```
 
 如果理解了上面的代码，那么寻找最大k个数就很简单了：
 
 ```JavaScript
function heap_sort(arr, len) {
  for (let i = parseInt(len / 2) - 1; i >= 0; i--) {
    heapify(arr, i, len - 1);
  }
  for (let i = len - 1; i >= len - k; i--) {
    swap(arr, 0, i); 
    heapify(arr, 0, i - 1); 
  }
}
```

以上两种方法都把时间复杂度从O(nlogn)优化了一些，变成了O(klogn)。

（ps：如有描述不当之处欢迎指正）

