---
layout:     post
title:      "树形DP：没有上司的舞会"
subtitle:   ""
date:       2024-08-25 18:35:00
author:     "西米屋花火"

catalog: true
header-img: "img/post-nana.jpg"
header-mask: 0.5
tags:
    - 动态规划
    - 树形DP
    - 树
    - 算法
---
## 题目描述

某大学有 ***n*** 个职员，编号为 ***1... n***。

他们之间有从属关系，也就是说他们的关系就像一棵以校长为根的树，父结点就是子结点的直接上司。

现在有个周年庆宴会，宴会每邀请来一个职员都会增加一定的快乐指数 ***r\_i***，但是呢，如果某个职员的直接上司来参加舞会了，那么这个职员就无论如何也不肯来参加舞会了。

所以，请你编程计算，邀请哪些职员可以使快乐指数最大，求最大的快乐指数。

## 输入格式

输入的第一行是一个整数 ***n***。

第 ***2*** 到第 ***(n + 1)*** 行，每行一个整数，第 ***(i+1)*** 行的整数表示 ***i*** 号职员的快乐指数 ***r\_i***。

第 ***(n + 2)*** 到第 ***2n*** 行，每行输入一对整数 ***l, k***，代表 ***k*** 是 ***l*** 的直接上司。

## 输出格式

输出一行一个整数代表最大的快乐指数。

## 样例 #1

### 样例输入 #1

    7
    1
    1
    1
    1
    1
    1
    1
    1 3
    2 3
    6 4
    7 4
    4 5
    3 5

### 样例输出 #1

    5

## 提示

#### 数据规模与约定

对于 ***100%*** 的数据，保证 ***1<= n <= 6 \* 10^3***，***-128 <= r\_i<= 127***，***1 <= l, k <= n***，且给出的关系一定是一棵树。

## 思路

#### 集合划分

*   f\[i]\[j]：只考虑以i为根节点的子树，最大的快乐值
*   j = true：i被邀请 j = false：i没被邀请

#### 状态转移

*   f\[i]\[1] = f\[j1]\[0] + f\[j2]\[0] + ...（j是i的子节点）
*   f\[i]\[0] = max(f\[j1]\[0], f\[j1]\[1]) + max(f\[j2]\[0], f\[j2]\[1]) + ...

## 题解

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int MAXN = 6e3 + 10;
//f[i][j]:只考虑以i为根节点的子树，最大的快乐值
//j = true：i被邀请 j = false：i没被邀请
int f[MAXN][2];
int n;
//存储快乐值
int happy[MAXN];
//邻接表存储，idx是用于存储的编号
//h[i]代表邻接表中第i个节点的头节点的存储编号
//e[i]代表存储编号i对应的值（即题目中的职员编号）
//ne[i]代表存储编号i的节点连接的下一个节点的存储编号
int h[MAXN], e[MAXN], ne[MAXN], idx;
//用于寻找没有父节点的节点，即整棵树的根节点，即题目中全体职员的最高上级
bool has_father[MAXN];

//a to b
//a和b都是值（即职员编号），而非存储编号
void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

//深度优先搜索树，u是职员编号
void dfs(int u) {
    //u被邀请，那么就先加上u的快乐值
    f[u][1] = happy[u];
    //循环遍历直到单链表的末尾
    for(int i = h[u]; i != -1; i = ne[i]) {
        //h、ne中存储的都是存储编号，用e来获取存储编号对应的职员编号
        int j = e[i];
        //因为遍历的顺序是从根节点开始到叶节点
        //所以只有递归的返回阶段才会进行状态转移方程的计算
        dfs(j);
        f[u][0] += max(f[j][0], f[j][1]);
        f[u][1] += f[j][0];
    }
}

int main() {
    //邻接表默认为空，则头节点的存储编号为-1，表示none
    memset(h, -1, sizeof h);
    cin >> n;
    for(int i = 1; i <= n; ++i) {
        cin >> happy[i];
    }
    for(int i = 0; i < n - 1; ++i) {
        int a, b;
        cin >> a >> b;
        add(b, a);
        has_father[a] = true;
    }
    //寻找整棵树的根节点
    int root = 1;
    while(has_father[root]) ++root;
    //从根节点开始搜索
    dfs(root);
    cout << max(f[root][0], f[root][1]);

}
```
