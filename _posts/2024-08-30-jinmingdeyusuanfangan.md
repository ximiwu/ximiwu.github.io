---
layout:     post
title:      "01背包问题：[NOIP2006 提高组] 金明的预算方案"
subtitle:   ""
date:       2024-08-30 22:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 动态规划
    - 线性DP
    - 01背包问题
    - 算法
---

## 题目描述

金明今天很开心，家里购置的新房就要领钥匙了，新房里有一间金明自己专用的很宽敞的房间。更让他高兴的是，妈妈昨天对他说：“你的房间需要购买哪些物品，怎么布置，你说了算，只要不超过 ***n*** 元钱就行”。今天一早，金明就开始做预算了，他把想买的物品分为两类：主件与附件，附件是从属于某个主件的，下表就是一些主件与附件的例子：

| 主件 | 附件 |
| :----------: | :----------: |
| 电脑 | 打印机，扫描仪 |
| 书柜 | 图书 |
| 书桌 | 台灯，文具 |
| 工作椅 | 无 |

如果要买归类为附件的物品，必须先买该附件所属的主件。每个主件可以有 ***0*** 个、***1*** 个或 ***2*** 个附件。每个附件对应一个主件，附件不再有从属于自己的附件。金明想买的东西很多，肯定会超过妈妈限定的 ***n*** 元。于是，他把每件物品规定了一个重要度，分为 ***5*** 等：用整数 ***1 ~ 5*** 表示，第 ***5*** 等最重要。他还从因特网上查到了每件物品的价格（都是 ***10*** 元的整数倍）。他希望在不超过 ***n*** 元的前提下，使每件物品的价格与重要度的乘积的总和最大。

设第 ***j*** 件物品的价格为 ***v_j***，重要度为 ***w_j***，共选中了 ***k*** 件物品，编号依次为 ***j_1,j_2,...,j_k***，则所求的总和为：

******v_{j_1} * w_{j_1}+v_{j_2} * w_{j_2}+ ... +v_{j_k} * w_{j_k}******

请你帮助金明设计一个满足要求的购物单。

## 输入格式

第一行有两个整数，分别表示总钱数 ***n*** 和希望购买的物品个数 ***m***。

第 ***2*** 到第 ***(m + 1)*** 行，每行三个整数，第 ***(i + 1)*** 行的整数 ***v_i***，***p_i***，***q_i*** 分别表示第 ***i*** 件物品的价格、重要度以及它对应的的主件。如果 ***q_i=0***，表示该物品本身是主件。

## 输出格式

输出一行一个整数表示答案。

## 样例 #1

### 样例输入 #1

```
1000 5
800 2 0
400 5 1
300 5 1
400 3 0
500 2 0
```

### 样例输出 #1

```
2200
```

## 提示

#### 数据规模与约定

对于全部的测试点，保证 ***1 <= n <= 3.2 * 10^4***，***1 <= m <= 60***，***0 <= v_i <= 10^4***，***1 <= p_i <= 5***，***0 <= q_i <= m***，答案不超过 ***2 * 10^5***。

NOIP 2006 提高组 第二题
## 思路

*   将每个主件极其对应的附件视为一组，把每一组当成一个大物品，用类似于01背包问题的做法
*   对于一个物品，传统的01背包对应两种决策：

    1.  选
    2.  不选
*   对于一个物品，本题对应五种决策：

    1.  只选主件
    2.  选主件、附件1
    3.  选主件、附件2
    4.  选主件、附件1、2
*   使用树来存储主件、附件，题解中使用的是邻接表实现

## 题解

#### 二维

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int MAXN = 65;
const int MAXNN = 3.2e4 + 5;
int money, m;
int v[MAXN], p[MAXN], q[MAXN];

//f[i][j]：考虑物品编号<=i的主件，预算为j的情况下，重要度最大值
int f[MAXN][MAXNN];

int head[MAXN], e[MAXN], ne[MAXN], idx;

//a to b
void add(int a, int b) {
    e[idx] = b, ne[idx] = head[a];
    head[a] = idx++;
}

int main() {

    memset(head, -1, sizeof head);
    cin >> money >> m;

    //i=0：主树的根，连接了各个主件
    //i!=0：物件节点
    for(int i = 1; i <= m; ++i) {
        cin >> v[i] >> p[i] >> q[i];
        add(q[i], i);
    }

    //记录上一次循环的节点
    int last_father = 0;

    for(int i = head[0]; i != -1; i = ne[i]) {
        //i是存储编号（节点编号），用e获取对应的物品编号
        int father = e[i];

        for(int j = 0; j <= money; ++j) {
            //能否选当前主件
            if(j >= v[father]) {
                f[father][j] = max(f[last_father][j], v[father] * p[father] + f[last_father][j - v[father]]);
                int child1 = -1, child2 = -1;

                //判断当前主件有没有第一个附件、第二个附件，并获取对应物品编号
                if(head[father] != -1) {
                    child1 = e[head[father]];
                    if(ne[head[father]] != -1) child2 = e[ne[head[father]]];
                }

                //选附件1
                if(child1 != -1 && j >= v[father] + v[child1]) {
                    f[father][j] = max(f[father][j],
                                       f[last_father][j - v[father] - v[child1]]
                                       + v[father] * p[father] + v[child1] * p[child1]);
                }
                //选附件2
                if(child2 != -1 && j >= v[father] + v[child2]) {
                    f[father][j] = max(f[father][j],
                                       f[last_father][j - v[father] - v[child2]]
                                       + v[father] * p[father] + v[child2] * p[child2]);
                }
                //选附件1、2
                if(child1 != -1 && child2 != -1 && j >= v[father] + v[child1] + v[child2]) {
                    f[father][j] = max(f[father][j],
                                       f[last_father][j - v[father] - v[child1] - v[child2]]
                                       + v[father] * p[father] + v[child1] * p[child1]
                                       + v[child2] * p[child2]);
                }


            }
            else f[father][j] = f[last_father][j];
        }
        last_father = father;

    }
    cout << f[last_father][money];

}


```

#### 一维

    #include <iostream>
    #include <algorithm>
    #include <cstring>
    #include <limits.h>
    using namespace std;

    const int MAXN = 65;
    const int MAXNN = 3.2e4 + 5;
    int money, m;
    int v[MAXN], p[MAXN], q[MAXN];

    int f[MAXNN];

    int head[MAXN], e[MAXN], ne[MAXN], idx;

    //a to b
    void add(int a, int b) {
        e[idx] = b, ne[idx] = head[a];
        head[a] = idx++;
    }

    int main() {
        memset(head, -1, sizeof head);
        //memset(f, -1, sizeof f);
        cin >> money >> m;

        for(int i = 1; i <= m; ++i) {
            cin >> v[i] >> p[i] >> q[i];
            add(q[i], i);
        }
        
        for(int i = head[0]; i != -1; i = ne[i]) {
            int father = e[i];
            for(int j = money; j >= v[father]; --j) {
                f[j] = max(f[j], v[father] * p[father] + f[j - v[father]]);
                int child1 = -1, child2 = -1;

                if(head[father] != -1) {
                    child1 = e[head[father]];
                    if(ne[head[father]] != -1) child2 = e[ne[head[father]]];
                }

                if(child1 != -1 && j >= v[father] + v[child1]) {
                    f[j] = max(f[j],
                                  f[j - v[father] - v[child1]]
                                  + v[father] * p[father] + v[child1] * p[child1]);
                }
                if(child2 != -1 && j >= v[father] + v[child2]) {
                    f[j] = max(f[j],
                                  f[j - v[father] - v[child2]]
                                  + v[father] * p[father] + v[child2] * p[child2]);
                }

                if(child1 != -1 && child2 != -1 && j >= v[father] + v[child1] + v[child2]) {
                    f[j] = max(f[j],
                                  f[j - v[father] - v[child1] - v[child2]]
                                  + v[father] * p[father] + v[child1] * p[child1]
                                  + v[child2] * p[child2]);



                }
            }

        }
        cout << f[money];

    }

