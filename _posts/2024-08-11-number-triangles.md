---
layout:     post
title:      "线性DP：[USACO1.5][IOI1994]数字三角形 Number Triangles"
subtitle:   ""
date:       2024-08-11 00:00:00
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

观察下面的数字金字塔。

写一个程序来查找从最高点到底部任意处结束的路径，使路径经过数字的和最大。每一步可以走到左下方的点也可以到达右下方的点。

![image](https://cdn.luogu.com.cn/upload/image_hosting/95pzs0ne.png)

在上面的样例中，从 ***7 \to 3 \to 8 \to 7 \to 5*** 的路径产生了最大权值。

## 输入格式

第一个行一个正整数 ***r***, 表示行的数目。

后面每行为这个数字金字塔特定行包含的整数。

## 输出格式

单独的一行, 包含那个可能得到的最大的和。

## 样例 #1

### 样例输入 #1

    5
    7
    3 8
    8 1 0
    2 7 4 4
    4 5 2 6 5

### 样例输出 #1

    30

## 提示

【数据范围】
对于 ***100%*** 的数据，***1<= r <= 1000***，所有输入在 ***\[0, 100]*** 范围内。

题目翻译来自NOCOW。

USACO Training Section 1.5

IOI1994 Day1T1



## 思路

#### 集合划分：

*   f\[i]\[j]：表示从第一行第一列走到第i行第i列的所有走法中数字和最大值

#### 状态转移：

*   f\[i]\[j] = max(f左上方，f右上方) + num\[i]\[j]

## 题解

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int MAXN = 1010;

int r;
int num[MAXN][MAXN];
int f[MAXN][MAXN];

int main() {
    cin >> r;
    for(int i = 1; i <= r; ++i) {
        for(int j = 1; j <= i; ++j) {
            cin >> num[i][j];
        }
    }


    f[1][1] = num[1][1];

    for(int i = 1; i <= r; ++i) {
        for(int j = 1; j <= i; ++j) {
            f[i][j] = max(f[i - 1][j - 1] + num[i][j], f[i - 1][j] + num[i][j]);
        }
    }

    int max_num = 0;
    for(int i = 1; i <= r; ++i) {
        max_num = max(max_num, f[r][i]);
    }

    cout << max_num;
    

}
```



