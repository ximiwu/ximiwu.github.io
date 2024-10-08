---
layout:     post
title:      "Games101：光线追踪"
subtitle:   ""
date:       2024-10-08 15:50:00
author:     "西米屋花火"
catalog: true
header-mask: 0.5
tags:
    - Games系列课程
    - Games101
    - graphics
---

## 光栅化下的阴影映射

![12_Page35](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/12_Page35.png)

首先，在点光源放置摄像机做一个只进行深度测试的光栅化，得到一张灰度图，记录了点光源可以照亮的像素对应的深度。

然后在人眼处放置摄像机，光栅化时，对于每个像素点，投影变换到点光源处，查询灰度图，如果人眼看的像素点更深，说明光照不到，就是阴影

![12_Page36](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/12_Page36.png)

![12_Page37](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/12_Page37.png)

缺点：

- 结果很脏，因为灰度图本身就是光栅化的结果
- 只有产生硬阴影比较方便

## 光线追踪

### Whitted-Style Ray Tracing  

![13_Page17](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/13_Page17.png)

对于每个像素，与眼睛连接形成射线射到第一个物体表面形成点1，在点1上递归地产生反射、折射的射线不断射到更多的点2、3、4....每个点都与点光源连线都能形成一条完整光路。把所有光路按一定比例累加得到像素点的颜色。

缺点：

- 只能处理镜面反射，没办法处理漫反射
- 没有全局光照（物体间的多次漫反射）

### 射线与三角形网格相交

![13_Page28](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/13_Page28.png)

射线方程由起点与方向向量定义

平面方程由平面上两点与法向量定义

设一个点又在射线上又在平面上就能解出t

![13_Page29](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/13_Page29.png)

更快的算法：利用三角形重心坐标，如果解出b1、b2、1-b1-b2都为正，说明交点在三角形内

### 射线与三角形网格相交加速

#### 包围盒

![13_Page36](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/13_Page36.png)

由6个参数x0、x1、y0、y1、z0、z1定义AABB

![13_Page37](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/13_Page37.png)

由x0、x1可以算出一组t_min、t_max。y、z同理。

当三个维度都被射线进入了，射线才进入了包围盒

当任意一个维度被射线离开了，射线就离开了包围盒

因此，如果t_min的最大值<t_max的最小值，说明射线穿过了包围盒，反之射线没有穿过包围盒。

#### Bounding Volume Hierarchy  

一个场景的三角形非常多，让射线与每个三角形都判断一次相交十分费时，因此利用包围盒将三角形分组，如果射线没有穿过某个包围盒，那么包围盒内的所有三角形都不用再判断

![14_Page30](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/14_Page30.png)

根节点是包含了所有三角形的包围盒

![14_Page33](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/14_Page33.png)

如果射线穿过父节点，那么递归地判断父节点的两个子节点，如果穿过子节点，那么递归地判断子节点的两个子节点......

### 辐射度量学

#### Radiant Energy and Flux  

![14_Page43](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/14_Page43.png)

radiant energy:电磁辐射的能量

radiant flux:每单位时间发射、反射、传播、吸收的能量

#### Radiant Intensity  

![14_Page47](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/14_Page47.png)

辐射强度：点光源每单位立体角发射的能量

#### Solid Angles 

![14_Page48](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/14_Page48.png)

二维角度定义：弧长除以半径

立体角定义：立体角在球上投出的曲面面积除以半径平方

#### 单位立体角

![14_Page49](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/14_Page49.png)

将立体角投到球面上的曲面视为长方形

#### Irradiance 

![15_Page9](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/15_Page9.png)

辐照度：每个单位面积接收到的能量

### ![15_Page10](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/15_Page10.png)

表面的irradiance与光线角度有关

#### Radiance  

![15_Page15](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/15_Page15.png)

辐射率：每单位立体角，每单位面积，发射、反射、传播、吸收的能量

![15_Page16](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/15_Page16.png)

辐射率：每单位立体角的辐照度

辐射率：每单位面积的辐射强度

#### BRDF

![15_Page22](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/15_Page22.png)

BRDF定义了有多少光从每个入射方向反射到每个出射方向

#### 渲染方程

![15_Page30](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/15_Page30.png)

人眼看到某个点发出的光的构成：这个点自己发出的光、这个点反射的光（可以是光源发出的光、也可以是其他物体发出的光）

### 概率论

#### 概率分布函数

![15_Page47](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/15_Page47.png)

#### 蒙特卡洛积分

![16_Page11](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page11.png)

对函数采样N次来得到积分的估计

### Path Tracing

![16_Page23](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page23.png)

对于渲染方程中的半球积分，可以用蒙特卡洛积分求近似

![16_Page29](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page29.png)

对于每个像素点，随机选择某个方向发出可能的路径，多次重复进行后按加权平均，也就是在进行蒙特卡洛积分

对于一段射线，当射到某个物体时，就在射到的点处进行随机选择方向产生下一段射线，不断递归地进行，各个射线组合形成了一段路径

![16_Page31](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page31.png)

wo即出射角，wi即入射角，wi是随机选择的（蒙特卡洛积分）

![16_Page35](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page35.png)

为了解决不停递归地产生射线的问题，设置一个概率使射线停止传播，通过将结果除以P的方法使期望保持不变

![16_Page36](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page36.png)

#### 对光源采样

![16_Page38](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page38.png)

由于蒙特卡洛积分时是均匀采样，当光源面积很小时，难以采样到光源。因此应该增加对光源的采样率。

![16_Page40](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page40.png)

原本是对某点为球心的半球进行积分，现在改为对光源积分

![16_Page41](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page41.png)

![16_Page43](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page43.png)

光源的贡献和其他物体反射的贡献各自单独计算后加起来

![16_Page44](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/16_Page44.png)

要判断光源能否直接打到p点

