---
layout:     post
title:      "记忆化搜索：最大食物链计数"
subtitle:   ""
date:       2024-08-27 17:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 动态规划
    - 记忆化搜索
    - 树
    - 算法
---
## 题目背景

你知道食物链吗？Delia 生物考试的时候，数食物链条数的题目全都错了，因为她总是重复数了几条或漏掉了几条。于是她来就来求助你，然而你也不会啊！写一个程序来帮帮她吧。

## 题目描述

给你一个食物网，你要求出这个食物网中最大食物链的数量。

（这里的“最大食物链”，指的是**生物学意义上的食物链**，即**最左端是不会捕食其他生物的生产者，最右端是不会被其他生物捕食的消费者**。）

Delia 非常急，所以你只有 ***1*** 秒的时间。

由于这个结果可能过大，你只需要输出总数模上 ***80112002*** 的结果。

## 输入格式

第一行，两个正整数 ***n、m***，表示生物种类 ***n*** 和吃与被吃的关系数 ***m***。

接下来 ***m*** 行，每行两个正整数，表示被吃的生物A和吃A的生物B。

## 输出格式

一行一个整数，为最大食物链数量模上 ***80112002*** 的结果。

## 样例 #1

### 样例输入 #1

    5 7
    1 2
    1 3
    2 3
    3 5
    2 5
    4 5
    3 4

### 样例输出 #1

    5

## 提示

【补充说明】

数据中不会出现环，满足生物学的要求。（感谢 @AKEE ）

## 题解

```cpp
#include <iostream>
#include <cstring>
using namespace std;

int n, m;
const int mod = 80112002;
const int MAXN = 5100;
const int MAXNN = 510000;

int h[MAXN], e[MAXNN], ne[MAXNN], idx;
unsigned long long f[MAXN];
bool has_father[MAXN];

//a to b
void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}


unsigned long long dp(int u) {
    if(f[u] != -1) return f[u];

    unsigned long long res = 0;
    for(int i = h[u]; i != -1; i = ne[i]) {
        res += dp(e[i]) % mod;
    }
    if(res == 0) res = 1;
    f[u] = res;
    return res;
}


int main() {
    memset(f, -1 ,sizeof f);
    memset(h, -1, sizeof h);

    cin >> n >> m;
    for(int i = 0; i < m; ++i) {
        int a, b;
        cin >> a >> b;
        has_father[b] = true;
        add(a, b);
    }

    unsigned long long res = 0;
    for(int i = 1; i <= n; ++i) {
        if(!has_father[i] && h[i] != -1) res += dp(i) % mod;
    }
    cout << res % mod;
}
```

