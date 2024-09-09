---
layout:     post
title:      "线性DP：[USACO03FALL] Cow Exhibition G"
subtitle:   ""
date:       2024-09-09 22:00:00
author:     "西米屋花火"
catalog: true
header-mask: 0.5
tags:
    - 动态规划
    - 线性DP
    - 算法
---

## 题目描述

奶牛想证明它们是聪明而风趣的。为此，贝西筹备了一个奶牛博览会，她已经对 ***N*** 头奶牛进行了面试，确定了每头奶牛的智商和情商。

贝西有权选择让哪些奶牛参加展览。由于负的智商或情商会造成负面效果，所以贝西不希望出展奶牛的智商之和小于零，或情商之和小于零。满足这两个条件下，她希望出展奶牛的智商与情商之和越大越好，请帮助贝西求出这个最大值。

## 输入格式

第一行：单个整数 ***N***，***1 <= N <= 400***。

第二行到第 ***N+1*** 行：第 ***i+1*** 行有两个整数：***S\_i*** 和 ***F\_i***，表示第 ***i*** 头奶牛的智商和情商，− ***1000 <= S\_i;F\_i <= 1000***。

## 输出格式

输出单个整数：表示情商与智商和的最大值。贝西可以不让任何奶牛参加展览，如果这样做是最好的，输出 ***0***。

## 样例 #1

### 样例输入 #1

    5
    -5 7
    8 -6
    6 -3
    2 1
    -8 -5

### 样例输出 #1

    8

## 提示

选择第一头，第三头，第四头奶牛，智商和为−5+6+2 = 3，情商和为7−3+1 = 5。再加入第二号奶牛可使总和提升到10，不过由于情商和变成负的了，所以是不允许的

## 思路

*   与简单的DP不同，需要考虑两个不同事物组合的最值，因此一个事物要放在下标里记录
*   状态转移类似于01背包问题
*   存在负数，所以要开双倍数组，并且注意处理

## 题解

#### 正常版

和01背包一样可以把i完全删掉，但我懒

不降维是肯定开不出数组的，太大

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int MAXN = 410;
const int MAXNN = 8e5 + 10;
const int INF = 4e5;

//f[i][j]：考虑前i头牛，情商为j的情况下智商最大值
int f[2][MAXNN];

int iq[MAXN];
int eq[MAXN];

int n;

int main() {
    cin >> n;
    for(int i = 1; i <= n; ++i) cin >> iq[i] >> eq[i];

    //无穷小的意思是不可能
    memset(f, -1 * 0x3f3f3f3f, sizeof f);
    //优先处理i=1
    //如果不选1号，那么IQ、EQ都是0
    f[1][INF] = 0;
    //如果选1号，填上对应值就好
    f[1][eq[1] + INF] = iq[1];


    int idx = 0;
    for(int i = 2; i <= n; ++i, idx ^= 1) {
        //由于有负数，需要开两倍数组
        //0~4e5: -4e5 ~ 0  4e5+1~8e5: 1 ~ 4e5
        for(int j = 0; j <= 8e5; ++j) {
            if(j >= eq[i] && j - eq[i] <= 8e5)
                f[idx][j] = max(f[!idx][j], f[!idx][j - eq[i]] + iq[i]);
            else f[idx][j] = f[!idx][j];
        }
    }

    idx ^= 1;
    int res = -1 * 0x3f3f3f3f;
    //注意IQ、EQ两项单独不能为负数
    for(int j = 4e5; j <= 8e5; ++j) {
        if(f[idx][j] >= 0) res = max(res, f[idx][j] + j - INF);
    }

    if(res > 0) cout << res;
    else cout << 0;

}
```

#### 脑残版（反面教材）

区别在于如何划分下标与实际值的对应情况，这是作者一开始写代码时用的办法，特别麻烦，特别容易出错

精髓在于，下标应该符合正常的单调递增，且能满足加减运算，例如从正数被减成了负数，不应该有额外的判断换算

脑残版需要一大堆判断换算，作者真的很脑残

最佳实践：把所有实际值加上区间的一半获得下标值

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int MAXN = 410;
const int MAXNN = 8e5 + 1010;
const int INF = 4e5;

//f[i][j]：考虑前i头牛，情商为j的情况下智商最大值
int f[2][MAXNN];

int iq[MAXN];
int eq[MAXN];

int n;

int main() {
    cin >> n;
    for(int i = 1; i <= n; ++i) cin >> iq[i] >> eq[i];

    //无穷小的意思是不可能
    memset(f, -1 * 0x3f3f3f3f, sizeof f);
    //优先处理i=1
    //如果不选1号，那么IQ、EQ都是0
    f[1][0] = 0;
    //如果选1号，填上对应值就好
    if(eq[1] > 0) f[1][eq[1]] = iq[1];
    else f[1][INF - eq[1]] = iq[1];


    int idx = 0;
    for(int i = 2; i <= n; ++i, idx ^= 1) {
        //由于有负数，需要开两倍数组
        //1~4e5 、4e5+1 ~ 8e5 : -1 ~ -4e5 、0
        for(int j = -4e5; j <= 4e5; ++j) {
            int j_idx1 = j;
            int j_idx2 = j - eq[i];
            //负数就换算
            if(j_idx1 < 0) j_idx1 = 4e5 - j_idx1;
            //如果从负数变成了正数就换算
            if(j > 4e5 && j_idx2 < 4e5 && j_idx2 >= 0) j_idx2 = 4e5 - j;
            //如果从一个正数超过了4e5
            if(j <= 4e5 && j_idx2 > 4e5) continue;
            //负数就换算
            if(j_idx2 < 0) j_idx2 = 4e5 - j_idx2;

            f[idx][j_idx1] = max(f[!idx][j_idx1], f[!idx][j_idx2] + iq[i]);
        }
    }

    idx ^= 1;

    int res = -1 * 0x3f3f3f3f;
    //注意IQ、EQ两项单独不能为负数
    for(int j = 0; j <= 4e5; ++j) {
        if(f[idx][j] >= 0) res = max(res, f[idx][j] + j);
    }

    if(res > 0) cout << res;
    else cout << 0;

}
```

