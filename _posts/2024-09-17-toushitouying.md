---
layout:     post
title:      "Games101：透视投影变换"
subtitle:   ""
date:       2024-09-17 02:10:00
author:     "西米屋花火"
catalog: true
header-mask: 0.5
tags:
    - Games系列课程
    - Games101
    - graphics
---

## 三维变换矩阵

![1](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-1.PNG)

![2](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-2.PNG)

![4](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-4.PNG)

## 镜头变换

变换效果：把镜头移到0,0,0，朝向-z

![3](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-3.PNG)

旋转矩阵是正交矩阵，由正交矩阵的性质，旋转矩阵的转置就是旋转矩阵的逆

先求由标准角度转到镜头当前的角度，转置就得到了想要的旋转矩阵

## 正交投影变换

认为镜头距离物体无限远，无限远处相交的两条线就是平行线

把视锥（长方体）映射为标准立方体

![5](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-5.PNG)

## 透视投影变换

它分为两步：

1. 将视锥映射为长方体，也就是变为正交投影变换前的状态，课程中没有给出明确名字，我称它为“挤压变换”
2. 进行正交投影变换

接下来介绍挤压变换

![6](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-6.PNG)

挤压变换就是要让视锥的上下底面变成同样的大小

![7](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-7.PNG)

由相似三角形可得比例关系

![8](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-8.PNG)

希望通过待定系数法求得挤压变换矩阵，所以写出初始坐标与结果坐标

![9](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-9.PNG)

通过待定系数法，可以求得变换矩阵的第一、二、四行

接下来，求变换矩阵的第三行

![10](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-10.PNG)

定义近平面：视锥靠近镜头的那个面，z坐标为n（第二幅图中定义了n）

定义远平面：视锥离镜头较远的那个面

由视锥的近平面透视投影变换后不变，可得变换矩阵第三行的前两个元素是0，且能得到A和B的一个方程。再得到一个方程就能解出AB

![11](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-11.PNG)

不难想出，透视投影变换时，视锥中的点的z坐标是不变的。

假设远平面的z坐标为f

由视锥的远平面的中心点变换后，x,y,z坐标都不变，为（0,0,f）可得关于AB的另一个方程，由此解出A和B

于是我们得到了挤压变换矩阵

![16](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-16.png)

先进行挤压变换，再进行正交投影变换，就得到了透视投影变换

于是我们得到了透视投影变换矩阵

![12](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-12.PNG)

画面比例：近平面长宽比

fov（视野范围）：近平面上下两边中点，与镜头位置连线的张角

![14](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-14.PNG)

由画面比例与fov，可以算出近平面的四边的坐标

![13](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-13.PNG)

至此，各种矩阵变换结束，接下来就是栅格化了。

## homework

![15](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/1-15.PNG)

```cpp
#include "Triangle.hpp"
#include "rasterizer.hpp"
#include <eigen3/Eigen/Eigen>
#include <iostream>
#include <opencv2/opencv.hpp>
#include <cmath>

constexpr double MY_PI = 3.1415926;

Eigen::Matrix4f get_view_matrix(Eigen::Vector3f eye_pos)
{
    Eigen::Matrix4f view = Eigen::Matrix4f::Identity();

    Eigen::Matrix4f translate;
    translate << 1, 0, 0, -eye_pos[0], 0, 1, 0, -eye_pos[1], 0, 0, 1,
        -eye_pos[2], 0, 0, 0, 1;

    view = translate * view;

    return view;
}

Eigen::Matrix4f get_model_matrix(float rotation_angle)
{
    //返回绕z轴旋转的变换矩阵
    Eigen::Matrix4f model = Eigen::Matrix4f::Identity();

    float rad = rotation_angle / 180.0 * MY_PI;

    model(0, 0) = std::cos(rad);
    model(1, 0) = std::sin(rad);
    model(0, 1) = -std::sin(rad);
    model(1, 1) = std::cos(rad);

    return model;
}

Eigen::Matrix4f get_projection_matrix(float eye_fov, float aspect_ratio,
                                      float zNear, float zFar)
{
	
    Eigen::Matrix4f projection = Eigen::Matrix4f::Identity();

    float half_fov_rad = eye_fov / 2 * MY_PI / 180;

    float top = - std::tan(half_fov_rad) * zNear;
    float bottom = - top;
    float right = aspect_ratio * top;
    float left = - right;

    Eigen::Matrix4f ortho = Eigen::Matrix4f::Identity();
    ortho(0, 0) = 2 / (right - left);
    ortho(1, 1) = 2 / (top - bottom);
    ortho(2, 2) = 2 / (zFar - zNear);
    
    Eigen::Matrix4f perspect = Eigen::Matrix4f::Zero();
    perspect(0, 0) = zNear;
    perspect(1, 1) = zNear;
    perspect(2, 2) = zNear + zFar;
    perspect(2, 3) = zNear * zFar;
    perspect(3, 2) = 1;

    projection = ortho * perspect;
    return projection;
}

int main(int argc, const char** argv)
{
    float angle = 0;
    bool command_line = false;
    std::string filename = "output.png";

    if (argc >= 3) {
        command_line = true;
        angle = std::stof(argv[2]); // -r by default
        if (argc == 4) {
            filename = std::string(argv[3]);
        }
    }

    rst::rasterizer r(700, 700);

    Eigen::Vector3f eye_pos = {0, 0, 5};

    std::vector<Eigen::Vector3f> pos{ {2, 0, -2}, {0, 2, -2}, {-2, 0, -2} };

    std::vector<Eigen::Vector3i> ind{ {0, 1, 2} };

    auto pos_id = r.load_positions(pos);
    auto ind_id = r.load_indices(ind);

    int key = 0;
    int frame_count = 0;

    if (command_line) {
        r.clear(rst::Buffers::Color | rst::Buffers::Depth);

        r.set_model(get_model_matrix(angle));
        r.set_view(get_view_matrix(eye_pos));
        r.set_projection(get_projection_matrix(45, 1, 0.1, 50));

        r.draw(pos_id, ind_id, rst::Primitive::Triangle);
        cv::Mat image(700, 700, CV_32FC3, r.frame_buffer().data());
        image.convertTo(image, CV_8UC3, 1.0f);

        cv::imwrite(filename, image);

        return 0;
    }

    while (key != 27) {
        r.clear(rst::Buffers::Color | rst::Buffers::Depth);

        r.set_model(get_model_matrix(angle));
        r.set_view(get_view_matrix(eye_pos));
        r.set_projection(get_projection_matrix(45, 1, 0.1, 50));

        r.draw(pos_id, ind_id, rst::Primitive::Triangle);

        cv::Mat image(700, 700, CV_32FC3, r.frame_buffer().data());
        image.convertTo(image, CV_8UC3, 1.0f);
        cv::imshow("image", image);
        key = cv::waitKey(10);

        std::cout << "frame count: " << frame_count++ << '\n';

        if (key == 'a') {
            angle += 10;
        }
        else if (key == 'd') {
            angle -= 10;
        }
    }

    return 0;
}

```

