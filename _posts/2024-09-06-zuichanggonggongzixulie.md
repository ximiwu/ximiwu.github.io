---
layout:     post
title:      "贪心：最长公共子序列"

subtitle:   ""
date:       2024-09-06 21:00:00
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

给出 ***1,2,...,n*** 的两个排列 ***P\_1*** 和 ***P\_2*** ，求它们的最长公共子序列。

## 输入格式

第一行是一个数 ***n***。

接下来两行，每行为 ***n*** 个数，为自然数 ***1,2,...,n*** 的一个排列。

## 输出格式

一个数，即最长公共子序列的长度。

## 样例 #1

### 样例输入 #1

    5
    3 2 1 4 5
    1 2 3 4 5

### 样例输出 #1

    3

## 提示

*   对于 ***50%*** 的数据， ***n \le 10^3***；
*   对于 ***100%*** 的数据， ***n \le 10^5***。

## 思路

*   两个序列都是自然数 1,2,…,*n* 的一个排列，即没有重复，且两个序列包含相同且相等的数
*   序列1：3 2 1 4 5 序列2：1 2 3 4 5

    *   现在赋予序列1顺序数
    *   序列1变为：ABCDE，即12345
    *   序列2则变为：CBADE，即32145
    *   此时，如果两个序列存在公共子序列，那么它一定是上升序列，因为序列1是上升序列
    *   问题被转化为求序列2的最长上升子序列长度

        *   利用贪心可以求上升子序列划分数最大值
        *   由Dilworth 定理
            *   反链的最长子序列=正链的序列最小划分数

            *   不升子序列的反链：上升子序列

            *   上升子序列的反链：不升子序列
        *   则问题被进一步转化为求序列2的不升子序列划分数最大值

## 题解

```cpp
#include <iostream>
using namespace std;

int n;
const int MAXN = 1e5 + 10;
int seq1[MAXN];
int seq2[MAXN];

int map[MAXN];

int q[MAXN];
int tail = -1;

//在队列中新增一个元素
void new_q(int i) {
    q[++tail] = i;
}


int main() {
    cin >> n;
    for(int i = 1; i <= n; ++i) cin >> seq1[i];
    for(int i = 1; i <= n; ++i) cin >> seq2[i];

    //key:题目中的序列值 value:新赋予的顺序数
    //对于第一个序列，把第一个元素赋予对应的顺序数
    //即seq[i] = i;
    //而原本的值用map的下标来存储
    for(int i = 1; i <= n; ++i) map[seq1[i]] = i;
    //用顺序数更新第二个序列
    for(int i = 1; i <= n; ++i) seq2[i] = map[seq2[i]];

    //特判队列为空的情况
    new_q(seq2[1]);

    //二分法的三件套
    int min, max, mid;
    for(int i = 2; i <= n; ++i) {
        min = 0;
        max = tail;
        //对于每个序列二元素，在队列中查找第一个比它大的元素并替换
        //自然就形成了单调递增的单调队列
        while(min <= max) {
            mid = (min + max) / 2;
            if(q[mid] > seq2[i]) {
                if(mid == 0 || q[mid - 1] < seq2[i]) {
                    q[mid] = seq2[i];
                    break;
                }
                else max = mid - 1;
            }
            else min = mid + 1;
        }
        //如果没找到，就在队列中新增一个元素
        if(min > max) new_q(seq2[i]);
    }

    //tail是下标
    cout << tail + 1;
}
```

## 相关题目

[贪心：\[NOIP1999 提高组\] 导弹拦截](https://ximiwu.github.io/2024/09/02/daodanlanjie/)
