---
layout:     post
title:      "区间DP：[NOI1995] 石子合并"
subtitle:   ""
date:       2024-08-10 23:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 动态规划
    - 区间DP
    - 算法
---

## 题目描述

在一个圆形操场的四周摆放 ***N*** 堆石子，现要将石子有次序地合并成一堆，规定每次只能选相邻的 ***2*** 堆合并成新的一堆，并将新的一堆的石子数，记为该次合并的得分。

试设计出一个算法, 计算出将 ***N*** 堆石子合并成 ***1*** 堆的最小得分和最大得分。

## 输入格式

数据的第 ***1*** 行是正整数 ***N***，表示有 ***N*** 堆石子。

第 ***2*** 行有 ***N*** 个整数，第 ***i*** 个整数 ***a\_i*** 表示第 ***i*** 堆石子的个数。

## 输出格式

输出共 ***2*** 行，第 ***1*** 行为最小得分，第 ***2*** 行为最大得分。

## 样例 #1

### 样例输入 #1

    4
    4 5 9 4

### 样例输出 #1

    43
    54

## 提示

***1<=N<=100***，***0<=a\_i<=20***。

## 思路

#### 集合划分：

*   f\[i]\[j]->从第i堆石子到第j堆石子合并的得分最值

#### 环形如何解决：

*   将环形拆为一排， 然后复制一遍

    如：ABCD -> ABCDABCD

    只需确保一次的枚举次数为4即可

    从A开始：ABCD，从B开始：BCDA，从C开始：CDAB，符合环形

#### 状态转移：

*   f\[i]\[j] = f\[i]\[k] + f\[k + 1]\[j] + 从i到j石子的和

## 解法

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int MAXN = 210;

int n;
int a[MAXN];
int f_min[MAXN][MAXN];
int f_max[MAXN][MAXN];

int main() {
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];//按排读入
    }
    for (int i = 1; i <= n; ++i) {
        a[i + n] = a[i];//复制一遍
    }

    for (int i = 1; i <= n * 2; ++i) {
        a[i] += a[i - 1];//转为前缀和（便于从i到j的求和）
    }
	//枚举长度的理由：如果单纯ij两层嵌套枚举，例如n=4,i=1,j=3,k=1时，f[1][3] = min(f[1][3], f[1][1] + f[2][3] + a[3] - a[0])
	//其中f[2][3]没有被计算过，因此不能使用
	//而枚举长度则使i、j同时变化，当到达上述情况时(i=1,j=3)，len已经是2了，f[2][3]早在上一层len=1时计算过，因此可以使用
    for (int len = 2; len <= n; ++len) {//len=1无意义
        for (int i = 1; i + len - 1 <= n * 2; ++i) {//j不能超过2n
            int j = i + len - 1;
            f_min[i][j] = 1e8;
            f_max[i][j] = -1e8;
            for (int k = i; k < j; ++k){//极端情况：当len=2时，k=i，f[i][i]=0，符合题意。
					//而k不能=j，因为f[j+1][j]无意义
                f_min[i][j] = min(f_min[i][j], f_min[i][k] + f_min[k + 1][j] + a[j] - a[i - 1]);
                f_max[i][j] = max(f_max[i][j], f_max[i][k] + f_max[k + 1][j] + a[j] - a[i - 1]);
            }
        }
    }
    int min_num = 1e8;
    int max_num = -1e8;
    for (int i = 1; i <= n; ++i) {//枚举不同的起点，找出最值
        min_num = min(min_num, f_min[i][i + n - 1]);
        max_num = max(max_num, f_max[i][i + n - 1]);
    }
    cout << min_num << endl << max_num;
}
```

