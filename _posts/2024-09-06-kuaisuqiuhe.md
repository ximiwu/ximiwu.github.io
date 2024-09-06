---
layout:     post
title:      "线性DP：快速求和"
subtitle:   ""
date:       2024-09-06 12:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 动态规划
    - 线性DP
    - 算法
---

注：虽然是黄题，但比大部分绿题的通过率还低

## 题目描述

给定一个数字字符串，用最小次数的加法让字符串等于一个给定的目标数字。每次加法就是在字符串的某个位置插入一个加号。在里面要的所有加号都插入后，就像做普通加法那样来求值。

例如，考虑字符串`12`，做 ***0*** 次加法，我们得到数字 ***12***。如果插入 ***1*** 个加号，我们得到 ***3***，因此，这个例子中，最少用 ***1*** 次加法就得到数字 ***3***。

再举一例，考虑字符串`303`和目标数字 ***6***，最佳方法不是`3+0+3`。而是`3+03`。能这样做是因为一个数的前导 ***0*** 不会改变它的大小。

## 输入格式

第一行：一个字符串 ***s***。

第二行：一个整数 ***n***。

## 输出格式

一行一个整数表示最少的加法次数让 ***s*** 等于 ***n***。如果怎么做都不能让 ***s*** 等于 ***n*** ，则输出 ***-1***。

## 样例 #1

### 样例输入 #1

    99999
    45

### 样例输出 #1

    4

## 提示

#### 数据规模与约定

对于 ***100%*** 的数据，保证 ***1<= len(s)<=40***，***1 <=q n<=10^5***。

## 思路

#### 集合划分

*   num\[x]\[y]：从x到y的字串对应的数字

<!---->

*   f\[i]\[j]：从1到i的子串，使求和为j的最小操作数

#### 状态转移

*   f\[i]\[j] = min(f\[k]\[j - num\[k + 1]\[i]]) + 1（1 <= k <= i - 1）

    *   k从1到i-1枚举，上式表示从k+1到i完全不操作，只考虑1到k操作
    *   因为f\[k]\[]本身意味着k与k+1之间没有加号
*   f\[i]\[num\[1]\[i]] = 0（意思是从1到i完全不操作）

## 题解

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

string s;
int n;
const int MAXN = 50;
const int MAXNN = 1e5 + 10;
unsigned long long num[MAXN][MAXN];
unsigned long long f[MAXN][MAXNN];

int main() {

    //可忽略
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);

    cin >> s >> n;

    int size = s.size();

    //计算num[][]
    for(int i = 1; i <= size; ++i) {
        for(int j = i; j <= size; ++j) {
            //由于unsigned long long 最多存18,446,744,073,709,551,616 数量级为10^19
            //本题的字符串长度可达40位，会溢出，因此当长度太长的时候，不进行转换，而是设为1e8极大值
            //但又因为可能出现前导零，例如0000000000000000001，此时会被认为长度太长而设为极大值，是我们不希望的
            //因此在计算长度前，应该先删除前导零

            //忽略前导零
            int fixed_i = i;
            while(s[fixed_i - 1] == '0' && fixed_i != j) fixed_i += 1;

            //判断长度是否过长
            if(j - fixed_i <= 6) num[i][j] = stoull(s.substr(fixed_i - 1, j - fixed_i + 1));
            else num[i][j] = 1e8;
        }
    }
    //设为极大值
    memset(f, 0x3f3f3f3f, sizeof f);

    for(int i = 1; i <= size; ++i) {
        //特判1到i完全不操作的情况
        //注意n的取值范围，超过n的忽略
        if(num[1][i] <= 1e5) f[i][num[1][i]] = 0;

        //枚举求和大小
        for(int k = 0; k <= n; ++k) {
            //j从后往前枚举，因为枝剪的判断条件是num[j + 1][i] <= n
            //j最多枚举到1，完全不操作的情况已经被特判
            for(int j = i - 1; j >= 1 && num[j + 1][i] <= n; --j) {
                if(k >= num[j + 1][i])
                    f[i][k] = min(f[i][k], f[j][k - num[j + 1][i]] + 1);
            }
        }
    }
    //如果f被更新过，就不会是极大值，也就会小于45（字符串最大长度40）
    if(f[size][n] < 45) {
        cout << f[size][n];
    }
    else cout << -1;
}
```

