---
layout:     post
title:      "贪心：[NOIP1999 提高组] 导弹拦截"
subtitle:   ""
date:       2024-09-02 00:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 贪心
    - 二分
    - 算法
---

## 题目描述

某国为了防御敌国的导弹袭击，发展出一种导弹拦截系统。但是这种导弹拦截系统有一个缺陷：虽然它的第一发炮弹能够到达任意的高度，但是以后每一发炮弹都不能高于前一发的高度。某天，雷达捕捉到敌国的导弹来袭。由于该系统还在试用阶段，所以只有一套系统，因此有可能不能拦截所有的导弹。


输入导弹依次飞来的高度，计算这套系统最多能拦截多少导弹，如果要拦截所有导弹最少要配备多少套这种导弹拦截系统。

## 输入格式

一行，若干个整数，中间由空格隔开。

## 输出格式

两行，每行一个整数，第一个数字表示这套系统最多能拦截多少导弹，第二个数字表示如果要拦截所有导弹最少要配备多少套这种导弹拦截系统。

## 样例 #1

### 样例输入 #1

```
389 207 155 300 299 170 158 65
```

### 样例输出 #1

```
6
2
```

## 提示

对于前 ***50\%*** 数据（NOIP 原题数据），满足导弹的个数不超过 ***10^4*** 个。该部分数据总分共 ***100*** 分。可使用***n^2*** 做法通过。
对于后 ***50\%*** 的数据，满足导弹的个数不超过 ***10^5*** 个。该部分数据总分也为 ***100*** 分。请使用 ***nlog n*** 做法通过。

对于全部数据，满足导弹的高度为正整数，且不超过 ***5* 10^4***。


此外本题开启 spj，每点两问，按问给分。

NOIP1999 提高组 第一题

---

***\text{upd 2022.8.24}***：新增加一组 Hack 数据。
## 思路

*   第一问本质上是求最长不升子序列
*   第二问本质上是求不升子序列最小划分数

##### 由Dilworth 定理

*   反链的最长子序列=正链的序列最小划分数
*   不升子序列的反链：上升子序列
*   上升子序列的反链：不升子序列

<!---->

*   第一问可转化为求上升子序列最小划分数
*   第二问可转化为求最长上升子序列

#### 动态规划（TLE）

*   可以求最长子序列
*   f\[i]：以第i项为末尾的子序列的最大长度
*   状态转移：f\[i] = max(f\[k] + 1), k < i (k是子序列的倒数第二个)

#### 贪心（正解）

*   可以求最小划分数
*   f\[i]:第i个子序列末尾的值

    ##### 上升子序列最小划分数计算过程：

    *   遍历父序列，当前指向父序列的值记为a，对于每个a，在f中查找第一个小于a的项并替换为a，如果查找失败，则在f中新增一项并赋值为a。
    *   遍历完成后，f中有多少项，子序列最小划分数就是多少。
    *   查找应当使用二分法，复杂度为logn

## 题解

#### 贪心

```cpp
#include <iostream>
using namespace std;

const int MAXN = 1e5 + 10;

int height[MAXN];
//f[i]：第i个系统末尾的导弹高度
//上升子序列
int f[MAXN];
//不升子序列
int f2[MAXN];

int main() {
    //这个处理输入的方式似乎仅限oj，clion一定要输入非法才会停止
    int n = 0;
    while(cin >> height[n]) {
        ++n;
    }

    //二分法的三个指针
    int mid_j;
    int max_j = 0;
    int min_j = 0;
    //用于保存f有几项
    int his_max_j = 0;

    f[0] = height[0];
    //第一项单独处理，从第二项开始遍历
    for(int i = 1; i < n; ++i) {
        //二分法寻找f中第一个小于height[i]的j，并把f[j]替换为height[j]
        //如果不使用二分，新增的hack数据会tle
        while(min_j <= max_j) {
            mid_j = (max_j + min_j) / 2;
            if(f[mid_j] < height[i]) {
                if(mid_j == 0 || f[mid_j - 1] >= height[i]) {
                    f[mid_j] = height[i];
                    break;
                }
                else max_j = mid_j - 1;
            }
            else min_j = mid_j + 1;
        }
        //如果没有找到，说明f中需要新增一项
        if(min_j > max_j) f[++his_max_j] = height[i];
        min_j = 0;
        max_j = his_max_j;

    }
    //j是下标
    cout << max_j + 1 << endl;


    max_j = 0;
    min_j = 0;
    his_max_j = 0;
    f2[0] = height[0];
    for(int i = 1; i < n; ++i) {
        while(min_j <= max_j) {
            mid_j = (max_j + min_j) / 2;
            if(f2[mid_j] >= height[i]) {
                if(mid_j == 0 || f2[mid_j - 1] < height[i]) {
                    f2[mid_j] = height[i];
                    break;
                }
                else max_j = mid_j - 1;
            }
            else min_j = mid_j + 1;
        }
        if(min_j > max_j) f2[++his_max_j] = height[i];
        min_j = 0;
        max_j = his_max_j;
    }
    cout << max_j + 1 << endl;


}
```

#### 贪心但不二分（TLE）

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int MAXN = 1e5 + 10;

int height[MAXN];
//f[i]：第i个系统末尾的导弹高度
//上升子序列
int f[MAXN];
//不升子序列
int f2[MAXN];

int res1, res2;

int main() {
    int n = 1;

    while(cin >> height[n]) {
        ++n;
    }
    n -= 1;

    for(int i = 1; i <= n; ++i) {
        f[i] = -1;
        f2[i] = -1;
    }
    for(int i = 1; i <= n; ++i) {
        for(int j = 1; j <= n; ++j) {
            if(f[j] == -1 || height[i] > f[j]) {
                f[j] = height[i];
                res1 = max(res1, j);
                break;
            }
        }
        for(int j = 1; j <= n; ++j) {
            if(f2[j] == -1 || height[i] <= f2[j]) {
                f2[j] = height[i];
                res2 = max(res2, j);
                break;
            }
        }

    }

    cout << res1 << endl << res2;


}
```

#### 动态规划（TLE）

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int MAXN = 1e5 + 10;

int height[MAXN];
//不升子序列
int f[MAXN];
//上升子序列
int f2[MAXN];

int main() {
    int n = 1;

    while(cin >> height[n]) {
        ++n;
    }
    n -= 1;

    for(int i = 1; i <= n; ++i) {
        f[i] = 1;
        f2[i] = 1;
    }

    for(int i = 2; i <= n; ++i) {
        for(int j = 1; j != i; ++j) {
            if(height[j] >= height[i]) {
                f[i] = max(f[i], f[j] + 1);
            }
            if(height[j] < height[i]) {
                f2[i] = max(f2[i], f2[j] + 1);
            }
        }
    }

    int res = 0;
    for(int i = 1; i <= n; ++i) {
        res = max(res, f[i]);
    }
    cout << res << endl;

    res = 0;
    for(int i = 1; i <= n; ++i) {
        res = max(res, f2[i]);
    }
    cout << res << endl;



}
```

