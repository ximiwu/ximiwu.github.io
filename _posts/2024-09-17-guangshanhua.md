---
layout:     post
title:      "Games101：光栅化"
subtitle:   ""
date:       2024-09-17 14:00:00
author:     "西米屋花火"
catalog: true
header-mask: 0.5
tags:
    - Games系列课程
    - Games101
    - graphics
---

## 定义

光栅化：把通过前面的变换得到的标准立方体的视锥画在屏幕上的过程。

## 第一步：屏幕变换，把标准立方体映射到屏幕上

![1](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/2-1.PNG)

![2](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/2-2.PNG)

## 第二步：三角形的离散化——采样

![3](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/2-3.PNG)

通过判断每个像素的中心点是否在三角形内（向量叉乘），决定这个像素是否显色。

仅仅只是离散化，会使屏幕画面里的三角形产生锯齿

## 第三步：抗锯齿

从数字信号的角度来看，抗锯齿本质上是先把原始信号进行模糊，再进行采样，从而减少采样后混频的数据。

课程中通过数字信号的角度解释的非常深入，这里只说结论。

![4](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/2-4.PNG)

对于每个像素点，如果能计算出三角形的占比，再对应颜色的透明度，就能实现每个像素点的“模糊”。

但是三角形的占比计算开销过大，因此发明MSAA技术：超采样抗锯齿技术

![6](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/2-6.PNG)

![5](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/2-5.PNG)

通过对单个像素点的进一步细分，也就是“超采样”，来得到三角形在每个像素点占比的近似值。

## 第四步：Z-Buffer

![7](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/2-7.PNG)

通过记录每个像素点的z坐标最大值，可以知道当前处理的像素点是否可见。

## Homework

![8](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/2-8.PNG)

```cpp
Eigen::Matrix4f get_projection_matrix(float eye_fov, float aspect_ratio, float zNear, float zFar)
{
    Eigen::Matrix4f projection = Eigen::Matrix4f::Identity();

    float half_fov_rad = eye_fov / 2 * MY_PI / 180;

    float top = - std::tan(half_fov_rad) * zNear;
    float bottom = - top;
    float right = aspect_ratio * top;
    float left = - right;

    Eigen::Matrix4f ortho_trans = Eigen::Matrix4f::Identity();
    ortho_trans(0, 3) = -(right + left) / 2;
    ortho_trans(1, 3) = -(top + bottom) / 2;
    ortho_trans(2, 3) = -(zFar + zNear) / 2;

    Eigen::Matrix4f ortho_scale = Eigen::Matrix4f::Identity();
    ortho_scale(0, 0) = 2 / (right - left);
    ortho_scale(1, 1) = 2 / (top - bottom);
    ortho_scale(2, 2) = 2 / (zFar - zNear);


    Eigen::Matrix4f perspect = Eigen::Matrix4f::Zero();
    perspect(0, 0) = zNear;
    perspect(1, 1) = zNear;
    perspect(2, 2) = zNear + zFar;
    perspect(2, 3) = zNear * zFar;
    perspect(3, 2) = 1;

    projection = ortho_scale * ortho_trans * perspect;
    return projection;
}
```

```cpp
//基本原理就是记录上一个遍历的叉乘结果z轴正负，与这一次叉乘结果是否一致
static bool insideTriangle(int x, int y, const Vector3f* _v)
{
    bool direction = false;
    for(int i = 0; i < 3; ++i) {
        Eigen::Vector3f vec1(_v[i].x() - x, _v[i].y() - y, 0);
        int j = i + 1;
        if(j == 3) j = 0;
        Eigen::Vector3f vec2(_v[i].x() - _v[j].x(), _v[i].y() - _v[j].y(), 0);
        bool result;
        if((vec1.cross(vec2))(2) > 0) result = true;
        else result = false;
        if(i != 0 && result != direction) return false;
        direction = result;
    }
    return true;
}
```

```cpp
void rst::rasterizer::rasterize_triangle(const Triangle& t) {
    auto v = t.toVector4();
    //bounding box
    float top = std::max(t.v[0].y(), std::max(t.v[1].y(), t.v[2].y()));
    float bottom = std::min(t.v[0].y(), std::min(t.v[1].y(), t.v[2].y()));
    float right = std::max(t.v[0].x(), std::max(t.v[1].x(), t.v[2].x()));
    float left = std::min(t.v[0].x(), std::min(t.v[1].x(), t.v[2].x()));

    float b_top, b_bottom, b_right, b_left;
    b_top = std::ceil(top) - 0.5;
    b_bottom = std::floor(bottom) + 0.5;
    b_right = std::ceil(right) - 0.5;
    b_left = std::floor(left) + 0.5;
    //遍历bounding box内所有像素
    for(float x = b_left; x <= b_right; ++x) {
        for(float y = b_bottom; y <= b_top; ++y) {
            if(insideTriangle(x, y, t.v)) {
                //通过插值，获得三角形内部像素深度值
                auto[alpha, beta, gamma] = computeBarycentric2D(x, y, t.v);
                float w_reciprocal = 1.0/(alpha / v[0].w() + beta / v[1].w() + gamma / v[2].w());
                float z_interpolated = alpha * v[0].z() / v[0].w() + beta * v[1].z() / v[1].w() + gamma * v[2].z() / v[2].w();
                z_interpolated *= w_reciprocal;

                int x_index = x - 0.5;
                int y_index = y - 0.5;
                //zbuffer 算法
                if(depth_buf[get_index(x_index, y_index)] > z_interpolated) {
                    depth_buf[get_index(x_index, y_index)] = z_interpolated;
                    Eigen::Vector3f pixel_point(x_index, y_index, 1);
                    set_pixel(pixel_point, t.getColor());
                }
            }
        }
    }
}
```

