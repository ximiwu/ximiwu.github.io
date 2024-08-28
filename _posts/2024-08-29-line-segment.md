---
layout:     post
title:      "线性DP：[TJOI2007] 线段"
subtitle:   ""
date:       2024-08-29 01:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 动态规划
    - 线性DP
    - 算法
---

## 题目描述

在一个 ***n \* n*** 的平面上，在每一行中有一条线段，第 ***i*** 行的线段的左端点是(i, L\_i)，右端点是(i, R\_i)。

你从 ***(1,1)*** 点出发，要求沿途走过所有的线段，最终到达 ***(n,n)*** 点，且所走的路程长度要尽量短。

更具体一些说，你在任何时候只能选择向下走一步（行数增加 ***1***）、向左走一步（列数减少 ***1***）或是向右走一步（列数增加 ***1***）。当然，由于你不能向上行走，因此在从任何一行向下走到另一行的时候，你必须保证已经走完本行的那条线段。

## 输入格式

第一行有一个整数 ***n***。

以下 ***n*** 行，在第 ***i*** 行（总第 ***(i+1)*** 行）的两个整数表示 ***L\_i*** 和 ***R\_i***。

## 输出格式

仅包含一个整数，你选择的最短路程的长度。

## 样例 #1

### 样例输入 #1

    6
    2 6
    3 4
    1 3
    1 2
    3 6
    4 5

### 样例输出 #1

    24

## 提示

我们选择的路线是

    (1, 1) (1, 6)
    (2, 6) (2, 3)
    (3, 3) (3, 1)
    (4, 1) (4, 2)
    (5, 2) (5, 6)
    (6, 6) (6, 4) (6, 6)

不难计算得到，路程的总长度是 ***24***。

对于 ***100%*** 的数据中，***n <= 2 \* 10^4***，***1 <= L\_i <= R\_i <= n***。

## 误区

*   如果以坐标为集合划分标准，则这个图是有环的，无法使用DP

## 思路

#### 思考过程

*   题目要求走完所有线段，先提出一个维度i，表示走完第i个线段
*   然而还不够，因为走完线段后坐标不确定，因此再提出一个维度j，j=0，表示停在线段左端，j=1，表示停在线段右端

#### 状态转移方程

*   f\[i]\[1] = min(f\[i-1]\[0] + |l\[i - 1] - l\[i]| + 1 + r\[i] - l\[i], f\[i-1]\[1] + |r\[i - 1] - l\[i]| + 1 + r\[i] - l\[i])
*   f\[i]\[0] = min(f\[i-1]\[0] + |l\[i - 1] - r\[i]| + 1 + r\[i] - l\[i], f\[i-1]\[1] + |r\[i - 1] - r\[i]| + 1 + r\[i] - l\[i])

## 题解

#### 二维

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <limits.h>
using namespace std;

const int MAXN = 2e4 + 10;

int n;
int l[MAXN], r[MAXN];
//f[i][0 or 1];
int f[MAXN][2];

int main() {
    memset(f, INT_MAX, sizeof f);


    cin >> n;
    for(int i = 1; i <= n; ++i) {
        cin >> l[i] >> r[i];
    }

    f[1][0] = r[1] - 1 + r[1] - l[1];
    f[1][1] = r[1] - 1;

    for(int i = 2; i <= n; ++i) {
        f[i][1] = min(f[i - 1][0] + abs(l[i - 1] - l[i]) + 1 + r[i] - l[i],
                      f[i - 1][1] + abs(r[i - 1] - l[i]) + 1 + r[i] - l[i]);
        f[i][0] = min(f[i - 1][0] + abs(l[i - 1] - r[i]) + 1 + r[i] - l[i],
                      f[i - 1][1] + abs(r[i - 1] - r[i]) + 1 + r[i] - l[i]);
    }

    cout << min(f[n][1] + n - r[n], f[n][0] + n - l[n]);

}
```

#### 一维

显然，状态方程左边全是第i层，右边全是第i-1层。降维后可以与原式等价

空间换时间不存在了ヾ(^▽^\*)))

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <limits.h>
using namespace std;

const int MAXN = 2e4 + 10;

int n;
int l[MAXN], r[MAXN];
//f[i][0 or 1];
int f[2];

int main() {
    memset(f, INT_MAX, sizeof f);


    cin >> n;
    for(int i = 1; i <= n; ++i) {
        cin >> l[i] >> r[i];
    }

    f[0] = r[1] - 1 + r[1] - l[1];
    f[1] = r[1] - 1;

    for(int i = 2; i <= n; ++i) {
        int f0 = f[0];
        int f1 = f[1];
        f[1] = min(f0 + abs(l[i - 1] - l[i]) + 1 + r[i] - l[i],
                      f1 + abs(r[i - 1] - l[i]) + 1 + r[i] - l[i]);
        f[0] = min(f0 + abs(l[i - 1] - r[i]) + 1 + r[i] - l[i],
                      f1 + abs(r[i - 1] - r[i]) + 1 + r[i] - l[i]);
    }

    cout << min(f[1] + n - r[n], f[0] + n - l[n]);

}
```

