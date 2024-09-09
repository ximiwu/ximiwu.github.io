---
layout:     post
title:      "数位统计DP：计数问题"
subtitle:   ""
date:       2024-08-21 00:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 动态规划
    - 数位统计DP
    - 算法
---
## 题目描述

给定两个整数 ***a*** 和 ***b***，求 ***a*** 和 ***b*** 之间的所有数字中 ***0 \~ 9*** 的出现次数。

例如，***a=1024，b=1032***，则 ***a*** 和 ***b*** 之间共有 ***9*** 个数如下：

`1024 1025 1026 1027 1028 1029 1030 1031 1032`

其中 `0` 出现 ***10*** 次，`1` 出现 ***10*** 次，`2` 出现 ***7*** 次，`3` 出现 ***3*** 次等等…

## 输入格式

        输入包含多组测试数据。

每组测试数据占一行，包含两个整数 ***a*** 和 ***b***。

当读入一行为 `0 0` 时，表示输入终止，且该行不作处理。

## 输出格式

        每组数据输出一个结果，每个结果占一行。

每个结果包含十个用空格隔开的数字，第一个数字表示 `0` 出现的次数，第二个数字表示 `1` 出现的次数，以此类推。

## 数据范围

        0 < a,b < 100000000

## 输入样例：

```
1 10
44 497
346 542
1199 1748
1496 1403
1004 503
1714 190
1317 854
1976 494
1001 1960
0 0

```

## 输出样例：

```
1 2 1 1 1 1 1 1 1 1
85 185 185 185 190 96 96 96 95 93
40 40 40 93 136 82 40 40 40 40
115 666 215 215 214 205 205 154 105 106
16 113 19 20 114 20 20 19 19 16
107 105 100 101 101 197 200 200 200 200
413 1133 503 503 503 502 502 417 402 412
196 512 186 104 87 93 97 97 142 196
398 1375 398 398 405 499 499 495 488 471
294 1256 296 296 296 296 287 286 286 247

```

## 思路

#### 集合划分：

*   cnt(n,i):从1到n，i出现的次数

#### 状态转移：

*   a到b = （1到b） - （1到a）+ a



```cpp
#include <iostream>
#include <cmath>
using namespace std;

int a, b;

//获取一个数总共有几位，如：get_length(760) = 3
int get_length(int n) {
    int res = 0;
    while(n) {
        n /= 10;
        ++res;
    }
    return res;
}

//统计从1 ~ n中i出现几次
int cnt(int n, int i) {
    const int d = get_length(n);
    int res = 0;
    //j代表当前在统计第j位是i的情况
    for(int j = 1; j <= d; ++j) {
        int p = pow(10, j - 1);
        //第j位左边的数，如xxxjyyy中的xxx
        int left = n / p / 10;
        //第j位右边的数
        int right = n % p;
        //第j位的数
        int cur_num = n / p % 10;

        //当i不为0时
        if(i) {
            //对于当前统计的数，当第j位左边的数小于left时，左边的数可取0~left-1,右边的数可取0~p-1
            //如：n = 12345, j = 3, left = 12, right = 45
            //若：统计的数为11y32,11<12,11实际上可取0~11，32实际上可取00~99

            res += left * p;

            //对于当前统计的数，当第j位左边的数等于left时
            //如：n = 12345, j = 3, 统计的数为123xx，xx可取00~45

            if(i == cur_num) res += right + 1;

            //如：n = 12345, j = 3, 统计的数为121xx，xx可取00~99

            else if (i < cur_num) res += p;
        }

        //当i等于0时，则left一定不能等于0
        else if(!i && left) {

            //对于当前统计的数，当第j位左边的数小于left时，左边的数可取1~left-1,右边的数可取0~p-1

            res += (left - 1) * p;

            //对于当前统计的数，当第j位左边的数等于left时
            if(i == cur_num) res += right + 1;
            else res += p;
        }

    }
    return res;
}


int main() {

    while(true) {
        cin >> a >> b;
        if(a == 0 && b == 0) break;
        if(a > b) swap(a, b);
        //使用类似于前缀和的思想
        for(int i = 0; i < 10; ++i) {
            cout << cnt(b, i) - cnt(a - 1, i) << " ";//注意，a的情况要被包括，不能被减去，所以是a - 1
        }
        cout << endl;
    }

}

```
