---
layout:     post
title:      "线性DP：[NOIP2000 提高组] 方格取数"
subtitle:   ""
date:       2024-09-07 17:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 动态规划
    - 线性DP
    - 算法
---

## 题目背景

NOIP 2000 提高组 T4

## 题目描述

设有 ***N \* N*** 的方格图 ***(N <= 9)***，我们将其中的某些方格中填入正整数，而其他的方格中则放入数字 ***0***。如下图所示（见样例）:

![]([https://cdn.luogu.com.cn/upload/image_hosting/0bpummja.png](https://pub-2cdda6f679704f75bf34b7caab2ad90c.r2.dev/%E6%96%B9%E6%A0%BC%E5%8F%96%E6%95%B0.png))

某人从图的左上角的 ***A*** 点出发，可以向下行走，也可以向右走，直到到达右下角的 ***B*** 点。在走过的路上，他可以取走方格中的数（取走后的方格中将变为数字 ***0***）。
此人从 ***A*** 点到 ***B*** 点共走两次，试找出 ***2*** 条这样的路径，使得取得的数之和为最大。

## 输入格式

输入的第一行为一个整数 ***N***（表示 ***N \* N*** 的方格图），接下来的每行有三个整数，前两个表示位置，第三个数为该位置上所放的数。一行单独的 ***0*** 表示输入结束。

## 输出格式

只需输出一个整数，表示 ***2*** 条路径上取得的最大的和。

## 样例 #1

### 样例输入 #1

    8
    2 3 13
    2 6  6
    3 5  7
    4 4 14
    5 2 21
    5 6  4
    6 3 15
    7 2 14
    0 0  0

### 样例输出 #1

    67

## 提示

数据范围：***1<= N<= 9***。

## 思路

#### 四维

*   f\[x1]\[y1]\[x2]\[y2]：路线一到达x1,y1，路线二到达x2,y2的情况下最大值

#### 三维

*   f\[i]\[y1]\[y2]：两个路线同时行进i步，路线一到达(i-y1+2,y1)，路线二到达(i-y2+2,y2)
*   f\[i]\[y1]\[y2] = max(f\[i - 1]\[y1 - 1]\[y2 - 1], f\[i - 1]\[y1]\[y2], f\[i - 1]\[y1 - 1]\[y2], f\[i - 1]\[y1]\[y2 - 1])

#### 二维

*   不难发现，三维的状态转移方程左边全是i，右边全是i-1，因此i可以被降掉

## 题解

#### 三维

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int MAXN = 15;

int map[MAXN][MAXN];
int n;

//f[i][y1][y2]：两个路线都走了i步，路线一终点坐标y1，路线二终点坐标y2
int f[MAXN][MAXN][MAXN];

int maxx(int a, int b, int c, int d) {
    return max(max(a, b), max(c, d));
}


int main() {
    cin >> n;

    while(true) {
        int a, b, c;
        cin >> a >> b >> c;
        if(a == 0 && b == 0 && c == 0) break;
        map[b][a] = c;
    }
    f[0][1][1] = map[1][1];
    int x1, x2;
    for(int i = 1; i <= 2 * n - 2; ++i) {
        for(int y1 = 1; y1 <= n; ++y1) {
            x1 = i - y1 + 2;
            if(x1 <= 0 || x1 > n) continue;
            for(int y2 = 1; y2 <= n; ++y2) {
                x2 = i - y2 + 2;
                if(x2 <= 0 || x2 > n) continue;

                f[i][y1][y2] = maxx(f[i - 1][y1 - 1][y2 - 1], f[i - 1][y1][y2],
                                    f[i - 1][y1 - 1][y2], f[i - 1][y1][y2 - 1]);

                if(y1 == y2) f[i][y1][y2] += map[x1][y1];
                else f[i][y1][y2] += map[x1][y1] + map[x2][y2];
//                cout << "f " << i << " " << y1 << " " << y2 << " = " << f[i][y1][y2] << endl;
            }
        }
    }
    cout << f[2 * n - 2][n][n];


}
```

#### 二维

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int MAXN = 10;

int map[MAXN][MAXN];
int n;

//f[i][y1][y2]：两个路线都走了i步，路线一终点坐标y1，路线二终点坐标y2
int f[2][MAXN][MAXN];

int maxx(int a, int b, int c, int d) {
    return max(max(a, b), max(c, d));
}

int main() {
    cin >> n;

    while(true) {
        int a, b, c;
        cin >> a >> b >> c;
        if(a == 0 && b == 0 && c == 0) break;
        map[b][a] = c;
    }

    f[1][1][1] = map[1][1];
    int x1, x2;
    bool idx = 0;
    for(int i = 1; i <= 2 * n - 2; ++i, idx ^= 1) {
        for(int y1 = 1; y1 <= n; ++y1) {
            x1 = i - y1 + 2;
            if(x1 <= 0 || x1 > n) continue;
            for(int y2 = 1; y2 <= n; ++y2) {
                x2 = i - y2 + 2;
                if(x2 <= 0 || x2 > n) continue;

                f[idx][y1][y2] = maxx(f[!idx][y1 - 1][y2 - 1], f[!idx][y1][y2],
                                    f[!idx][y1 - 1][y2], f[!idx][y1][y2 - 1]);

                if(y1 == y2) f[idx][y1][y2] += map[x1][y1];
                else f[idx][y1][y2] += map[x1][y1] + map[x2][y2];
            }
        }
    }
    idx ^= 1;
    cout << f[idx][n][n];


}
```

