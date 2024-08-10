---
layout:     post
title:      "线性DP：最长上升子序列"
subtitle:   ""
date:       2024-08-11 00:20:00
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

这是一个简单的动规板子题。

给出一个由 ***n(n<= 5000)*** 个不超过 ***10^6*** 的正整数组成的序列。请输出这个序列的**最长上升子序列**的长度。

最长上升子序列是指，从原序列中**按顺序**取出一些数字排在一起，这些数字是**逐渐增大**的。

## 输入格式

第一行，一个整数 ***n***，表示序列长度。

第二行有 ***n*** 个整数，表示这个序列。

## 输出格式

一个整数表示答案。

## 样例 #1

### 样例输入 #1

    6
    1 2 4 1 3 4

### 样例输出 #1

    4

## 提示

分别取出 ***1***、***2***、***3***、***4*** 即可。

## 思路：

#### 集合划分：

*   f\[i]：表示从1到i的最长上升子序列长度

#### 状态转移：

*   f\[i] = max(1, f\[k] + 1)
*   (num\[k] < num\[i], 1 <= k < i)
## 题解
```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int MAXN = 5010;

int n;
int num[MAXN];
int f[MAXN];

int main() {
    cin >> n;
    for(int i = 1; i <= n; ++i) {
        cin >> num[i];
    }

    for(int i = 1; i <= n; ++i) {
        f[i] = 1;
        for(int k = 1; k <= i - 1; ++k) {
            if(num[k] < num[i]) {
                f[i] = max(f[k] + 1, f[i]);

            }
        }
    }

    int max_num = 0;
    for(int i = 1; i <= n; ++i) {
        max_num = max(max_num, f[i]);
    }

    cout << max_num;

}
```

