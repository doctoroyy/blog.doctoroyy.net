---
title: 最短路之Floyd算法
date: 2019-08-18 23:25:07
tags: 图论
categories: 算法
---

Floyd算法主要用来解决多源最短路径（任意两点间的最短路径）问题，本身基于动态规划，时间复杂度为O(n^3)。

假设两个点i，j，那么从i到j路径最短只有两种情况：

1. 从**i**直接到**j**


2. 从**i**出发经过某一个或某几个中间节点到达**j**

<!-- more -->

  假设有n个点，dis[i][j]表示i到j的最短距离，任意两点间的距离初始化为无穷大
  ```c++
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (i == j) dis[i][j] = 0;
      else dis[i][j] = INF;
    }
  }
  
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      dis[i][j] = i点到j点的距离 
    }
  }
  
  // 从i到j经过点1，如果经过点1时路径更短，则更新数组
  
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (dis[i][1] + dis[1][j] < dis[i][j]) {
        dis[i][j] = dis[i][1] + dis[1][j];
      }
    }
  }
  
  //接下来尝试经过点2
  
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (dis[i][2] + dis[2][j] < dis[i][j]) {
        dis[i][j] = dis[i][2] + dis[2][j];
      }
    }
  }
  ```
  
一共有n个点，所以一共可以尝试n次...

因此最终可以写成这样：

 ```c++
  for (int k = 1; k <= n; k++) {
    for (int i = 1; i <= n; i++) {
      for (int j = 1; j <= n; j++) {
        if (dis[i][k] + dis[k][j] < dis[i][j]) {
          dis[i][j] = dis[i][k] + dis[k][j];
        }
      }
    }
  }
 ```
 
这个算法理解上并不难，核心代码也就是这几行

接下来用Floyd算法解决一道例题：




#### 题目描述

小猫在研究有向图。小猫在研究联通性。

给定一张N个点，M条边的有向图，问有多少点对(u,v)(u<v)，满足u能到达v且v也能到达u。

输入描述:
```
第一行两个正整数N,M，表示点数与边数。接下来M行，第i行两个正整数ui,vi，表示一条从ui到vi的边，保证ui≠vi。
```
输出描述:
```
一行一个整数，表示点对数量。
```
示例1

```

3 3
1 2
2 3
3 2
```
输出
```
1
```
备注:
1≤N≤300,1≤M≤N(N−1)

思路：

Floyd算法跑一遍求出任意两点间的距离，然后判断dis[i][j]与dis[j][i]是否同时可达（单单这道题的话其实用Floyd算法并不是最优解）


```c++
#include<iostream>
using namespace std;
const int inf = 0x3f3f3f3f, maxn = 300 + 10;
int dis[maxn][maxn], vis[maxn][maxn];
int main() {
  for (int i = 0; i < maxn; i++) {
    for (int j = 0; j < maxn; j++) {
      if (i == j) dis[i][j] = 0;
      else dis[i][j] = inf;
    }
  }
  int n, m;
  cin >> n >> m;
  for (int i = 0; i < m; i++) {
    int a, b;
    cin >> a >> b;
    dis[a][b] = 1;
  }
  for (int k = 1; k <= n; k++) {
    for (int i = 1; i <= n; i++) {
      for (int j = 1; j <= n; j++) {
        if (dis[i][k] + dis[k][j] < dis[i][j] && dis[i][k] != inf && dis[k][j] != inf) {
          dis[i][j] = dis[i][k] + dis[k][j];
        }
      }
    }
  }
  int cnt = 0;
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (i == j) continue;
      if (vis[i][j] == 0 && vis[j][i] == 0 && dis[i][j] != inf && dis[j][i] != inf) {
        cnt++;
        vis[i][j] = vis[j][i] = 1;
      }
    }
  }
  cout << cnt;
  return 0;
}
```