---
title: 2019 ICPC国际大学生程序设计竞赛中国邀请赛（南昌）  H. Coloring Game
date: 2019-04-21 23:08:50
categories: ICPC
tags: 刷题
---

David has a white board with 2 \times N2×N grids.He decides to paint some grids black with his brush.He always starts at the top left corner and ends at the bottom right corner, where grids should be black ultimately.

<!--more-->

Each time he can move his brush up(↑), down(↓), left(←), right(→), left up(↖), left down(↙), right up(↗), right down (↘) to the next grid.

For a grid visited before,the color is still black. Otherwise it changes from white to black.

David wants you to compute the number of different color schemes for a given board. Two color schemes are considered different if and only if the color of at least one corresponding position is different.

### Input
One line including an integer n(0<n \le 10^9)n(0<n≤109)

### Output
One line including an integer, which represent the answer \bmod 1000000007mod1000000007

### 样例输入1
```
2
```

### 样例输出1
```
4
```
### 样例解释1
![image](https://res.jisuanke.com/img/upload/20190416/39d257b9b54ce6d5c5b6224693a2c0fb51cc56a3.png)

### 样例输入2
```
3
```
### 样例输出2
```
12
```
样例解释2
![image](https://res.jisuanke.com/img/upload/20190416/0e47d8da862d4bb0b40d0c86c8515157adacf052.png)


题目意思是说有一个2*N的网格，从左上角涂色，涂到右下角为止，可以往旁边8个方向扩散，问一共有多少种不同的涂色方案，结果对1e9+7取模

拿到这道题第一反应就是打表，试试看能不能发现某些规律

所以可以先来个暴力
```C++
#include<iostream>
#include<vector>
#include<set>
#include<cstring>
using namespace std;
const int maxn = 100;
set<vector<int> > sset;
int n;
int dir[8][2] = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}, {1, 1}, {1, -1}, {-1, 1}, {-1, -1}};
int maze[2][maxn];
int valid(int x, int y) {
  return x >= 0 && x < 2 && y >= 0 && y < n;
}
void dfs(int x, int y) {
  vector<int> v;
  if (x == 1 && y == n - 1) {
    for (int i = 0; i < 2; i++) {
      for (int j = 0; j < n; j++) {
        v.push_back(maze[i][j]);
      }
    }
    sset.insert(v);
    return;
  }
  for (int i = 0; i < 8; i++) {
    int nx = x + dir[i][0], ny = y + dir[i][1];
    if (valid(nx, ny) && !maze[nx][ny]) {
      maze[nx][ny] = 1;
      dfs(nx, ny);
      maze[nx][ny] = 0;
    }
  }
}
int main() {
  for (n = 1; n <= 50; n++) {
    sset.clear();
    memset(maze, 0, sizeof(maze));
    maze[0][0] = 1;
    dfs(0, 0);
    cout << sset.size() << endl;
  }
  return 0;
}
```
每产生一种方案就往vector容器里存，再通过set去重，发现结果是:
``` 
1
4
12
36
108
324
972
2916
8738
```
所以可以大胆推测，从n = 2开始，结果构成一个首项为4，公比为3的等比数列，所以终于可以用快速幂取模解决了

```C++
#include<iostream>
using namespace std;
const int MOD = 1e9 + 7;
long long quick_pow(int n, int k) {
  long long res = 1, base = n;
  while (k) {
    if (k & 1) res = res * base % MOD;
    base = base * base % MOD;
    k >>= 1;
  }
  return res;
}
int main() {
  int n;
  cin >> n;
  if (n == 1) cout << "1";
  else cout << 4 * quick_pow(3, n - 2) % MOD;
  return 0;
}
```
