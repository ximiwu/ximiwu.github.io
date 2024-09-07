---
layout:     post
title:      "贪心：[NOIP2004 提高组] 合唱队形"
subtitle:   ""
date:       2024-09-07 22:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 贪心
    - 算法
---

## 题目描述

***n*** 位同学站成一排，音乐老师要请其中的 ***n-k*** 位同学出列，使得剩下的 ***k*** 位同学排成合唱队形。

合唱队形是指这样的一种队形：设 ***k*** 位同学从左到右依次编号为 ***1,2,*** … ***,k***，他们的身高分别为 ***t\_1,t\_2,*** … ***,t\_k***，则他们的身高满足 ***t\_1< \cdots \<t\_i>t\_{i+1}>*** … ***>t\_k(1<= i<= k)***。

你的任务是，已知所有 ***n*** 位同学的身高，计算最少需要几位同学出列，可以使得剩下的同学排成合唱队形。

## 输入格式

共二行。

第一行是一个整数 ***n***（***2<= n<=100***），表示同学的总数。

第二行有 ***n*** 个整数，用空格分隔，第 ***i*** 个整数 ***t\_i***（***130<= t\_i<=230***）是第 ***i*** 位同学的身高（厘米）。

## 输出格式

一个整数，最少需要几位同学出列。

## 样例 #1

### 样例输入 #1

    8
    186 186 150 200 160 130 197 220

### 样例输出 #1

    4

## 提示

对于 ***50%*** 的数据，保证有 ***n <= 20***。

对于全部的数据，保证有 ***n <= 100***。

## 思路

*   正着求最长上升子序列，倒着求最长上升子序列（正着看就是下降）
*   由长度可以算出需要移除多少人

注：本题解假设你已经看完

*   [贪心：\[NOIP1999 提高组\] 导弹拦截](https://ximiwu.github.io/2024/09/02/daodanlanjie/)
*   [贪心：最长公共子序列](https://ximiwu.github.io/2024/09/06/zuichanggonggongzixulie/)

## 题解

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

int n;

const int MAXN = 110;
int height[MAXN];

int tail = -1;
int q[MAXN];
int increase_length[MAXN];
int decrease_length[MAXN];

int mid, low, high;

void q_add(int i) {
    q[++tail] = height[i];
}

int main() {
    cin >> n;
    for(int i = 1; i <= n; ++i) {
        cin >> height[i];
    }

    //求不升最小划分数
    q_add(1);
    increase_length[1] = 1;

    for(int i = 2; i <= n; ++i) {
        low = 0, high = tail;
        //找第一个比自己大的
        while(low <= high) {
            mid = (low + high) / 2;
            if(q[mid] == height[i]) break;
            if(q[mid] < height[i]) low = mid + 1;
            else {
                if(mid == 0 || q[mid - 1] <= height[i]) {
                    q[mid] = height[i];
                    break;
                }
                else high = mid - 1;
            }
        }
        if(low > high) q_add(i);
        increase_length[i] = tail + 1;
    }

    //倒着看的不升最小划分数，正着看就是不降最小划分数
    //只在for语句修改即可，其他的复制
    tail = -1;
    q_add(n);
    decrease_length[n] = 1;

    for(int i = n - 1; i >= 1; --i) {
        low = 0, high = tail;
        //找第一个比自己大的
        while(low <= high) {
            mid = (low + high) / 2;
            if(q[mid] == height[i]) break;
            if(q[mid] < height[i]) low = mid + 1;
            else {
                if(mid == 0 || q[mid - 1] <= height[i]) {
                    q[mid] = height[i];
                    break;
                }
                else high = mid - 1;
            }
        }
        if(low > high) q_add(i);
        decrease_length[i] = tail + 1;
    }

    int res = 1e4;
    for(int i = 1; i <= n; ++i) {
        res = min(res, 1 + n - increase_length[i] - decrease_length[i]);
    }
    cout << res;

}
```



