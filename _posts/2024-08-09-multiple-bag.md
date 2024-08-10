---
layout:     post
title:      "动态规划：多重背包问题-二进制优化"
subtitle:   ""
date:       2024-08-09 02:00:00
author:     "西米屋花火"
catalog: true
header-img: "img/post-nana.jpg"
tags:
    - 动态规划
    - 算法
---

## 题面
![image](https://ximiwu.github.io/img/post-2024-08-09-multiple-bag.png)


## 暴力解法（TLE）

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
 
int N, V;
 
const int MAXN = 110;
int v[MAXN], w[MAXN], s[MAXN];
int f[MAXN][MAXN];
 
 
int main() {
    cin >> N >> V;
    for(int i = 1; i <= N; ++i) {
        cin >> v[i] >> w[i] >> s[i];
    }
    
    for(int i = 1; i <= N; ++i) {
        for(int j = 0; j <= V; ++j) {
            for(int k = 0; k <= s[i] && j >= v[i] * k; ++k) {
                f[i][j] = max(f[i][j], f[i - 1][j - k * v[i]] + w[i] * k);
            }
        }
    }
    cout << f[N][V];
 
}
```

## 二进制优化：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
 
int N, V;
 
const int MAXN = 11010;
int v[MAXN], w[MAXN];
int f[MAXN];
int m = 1;
 
int main() {
    cin >> N >> V;
    for(int i = 1; i <= N; ++i) {
        int volume, weight, sum;
        cin >> volume >> weight >> sum;
 
        for(int cnt = 1; sum >= cnt; cnt *= 2) {
            v[m] = volume * cnt;
            w[m] = weight * cnt;
            ++m;
            sum -= cnt;
        }
 
        if(sum > 0) {
            v[m] = volume * sum;
            w[m] = weight * sum;
            ++m;
        }
    }
    
    for(int i = 1; i <= m - 1; ++i) {
        for(int j = V; j >= v[i]; --j) {
            f[j] = max(f[j], f[j - v[i]] + w[i]);
        }
    }
    cout << f[V];
 
}
```

## 优化思路：

原理：将一种物品按数量拆分为1、2、4、8、16...为一组，视一组为一个新物品，转化为[01背包问题](https://so.csdn.net/so/search?q=01%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%98\&spm=1001.2101.3001.7020)

为什么可行：

&#x9;暴力解法：在给定的一种物品下，需要枚举选这种物品0、1、2、3...个时的情况，时间复杂度为n\*k

&#x9;二进制优化：

&#x9;	1 + 2 = 3，即1、2、3都可以表示

&#x9;	1 + 4 = 5， 2 + 4 = 6， 3 + 4 = 7，即4、5、6、7、8都可以表示

&#x9;	以此类推，1到k都可以表示，优化后每个新物品都只有一件，k降为log(2)k

