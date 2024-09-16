---
layout:     post
title:      "Games101：2D矩阵变换"
subtitle:   ""
date:       2024-09-17 02:00:00
author:     "西米屋花火"
catalog: true
header-mask: 0.5
tags:
    - Games系列课程
    - Games101
    - graphics
---

## notes

![1](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/0-1.PNG)

![2](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/0-2.PNG)

## homework

        给定一个点 P=(2,1), 将该点绕原点先逆时针旋转 45◦，再平移 (1,2), 计算出 变换后点的坐标（要求用齐次坐标进行计算）。

```cpp
#include<cmath>
#include<eigen3/Eigen/Core>
#include<eigen3/Eigen/Dense>
#include<iostream>
using namespace Eigen;
const float PI = 3.1415926;

int main(){
    Vector3f p(2, 1, 1);
    Matrix3f rotator;
    float angle = 45.0;
    float rad = angle / 180.0 * PI;
    rotator <<
            std::cos(rad), -1 * std::sin(rad), 0,
            std::sin(rad), std::cos(rad), 0,
            0, 0, 1;
    Matrix3f translator;
    translator <<
               1, 0, 1,
            0, 1, 2,
            0, 0, 1;
    //矩阵连乘运算顺序从右到左
    Vector3f pnew = translator * rotator * p;
    std::cout << pnew;
}
```
