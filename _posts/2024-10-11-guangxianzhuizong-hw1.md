---
layout:     post
title:      "Games101：光线追踪-作业(1)"
subtitle:   ""
date:       2024-10-11 11:00:00
author:     "西米屋花火"
catalog: true
header-mask: 0.5
tags:
    - Games系列课程
    - Games101
    - graphics
---

在这部分的课程中，我们将专注于使用光线追踪来渲染图像。在光线追踪中最重要的操作之一就是找到光线与物体的交点。一旦找到光线与物体的交点，就可以执行着色并返回像素颜色。在这次作业中，我们需要实现两个部分：光线的生成和光线与三角的相交。本次代码框架的工作流程为：

1. 从 main 函数开始。我们定义场景的参数，添加物体（球体或三角形）到场景中，并设置其材质，然后将光源添加到场景中。
2. 调用 Render(scene) 函数。在遍历所有像素的循环里，生成对应的光线并将返回的颜色保存在帧缓冲区（framebuffer）中。在渲染过程结束后，帧缓冲区中的信息将被保存为图像。
3. 在生成像素对应的光线后，我们调用 CastRay 函数，该函数调用 trace 来查询光线与场景中最近的对象的交点。
4. 然后，我们在此交点执行着色。我们设置了三种不同的着色情况，并且已经为你提供了代码。

你需要修改的函数是：

•  **Renderer.cpp** **中的** **Render()**：这里你需要为每个像素生成一条对应的光线，然后调用函数 castRay() 来得到颜色，最后将颜色存储在帧缓冲区的相应像素中。

•  **Triangle.hpp** **中的** **rayTriangleIntersect()**: v0, v1, v2 是三角形的三个顶点，orig 是光线的起点，dir 是光线单位化的方向向量。tnear, u, v 是你需要使用我们课上推导的 Moller-Trumbore 算法来更新的参数。

```cpp
void Renderer::Render(const Scene& scene)
{
    std::vector<Vector3f> framebuffer(scene.width * scene.height);

    float scale = std::tan(deg2rad(scene.fov * 0.5f));
    float imageAspectRatio = scene.width / (float)scene.height;

    // Use this variable as the eye position to start your rays.
    Vector3f eye_pos(0);
    int m = 0;
    for (int j = 0; j < scene.height; ++j)
    {
        for (int i = 0; i < scene.width; ++i)
        {
            // generate primary ray direction
            float x = ((i + 0.5) / scene.width - 0.5) * 1 * 2 * scale * imageAspectRatio;
            float y = -((j + 0.5) / scene.height - 0.5) * 1 * 2 * scale;
            // TODO: Find the x and y positions of the current pixel to get the direction
            // vector that passes through it.
            // Also, don't forget to multiply both of them with the variable *scale*, and
            // x (horizontal) variable with the *imageAspectRatio*            

            Vector3f dir = normalize(Vector3f(x, y, -1)); // Don't forget to normalize this direction!

            framebuffer[m++] = castRay(eye_pos, dir, scene, 0);
        }
        UpdateProgress(j / (float)scene.height);
    }

    // save framebuffer to file
    FILE* fp = fopen("binary.ppm", "wb");
    (void)fprintf(fp, "P6\n%d %d\n255\n", scene.width, scene.height);
    for (auto i = 0; i < scene.height * scene.width; ++i) {
        static unsigned char color[3];
        color[0] = (char)(255 * clamp(0, 1, framebuffer[i].x));
        color[1] = (char)(255 * clamp(0, 1, framebuffer[i].y));
        color[2] = (char)(255 * clamp(0, 1, framebuffer[i].z));
        fwrite(color, 1, 3, fp);
    }
    fclose(fp);    
}
```

注意i、j全都是正数，而eye_pos是0，0，0，意味着像素点的坐标有正有负，需要进行转换

以计算x为例，

(i + 0.5) / scene.width 是当前像素中心点位置占scene.width的比例，值域为(0, 1)

(i + 0.5) / scene.width - 0.5 是将比例值域从(0, 1)转化成(-0.5, 0.5)后中心点位置占比（相当于eye_pos从scene的左下角移到了scene的正中心，这样才符合eye_pos0，0，0）

再*2比例值域就变成(-1, 1)，方便直接使用

scale就是fov角的一半的tan值

*1是因为眼睛到scene平面距离为1（由Vector3f dir = normalize(Vector3f(x, y, -1))可知）

imageAspectRatio = scene.width / (float)scene.height

```cpp
bool rayTriangleIntersect(const Vector3f& v0, const Vector3f& v1, const Vector3f& v2, const Vector3f& orig,
                          const Vector3f& dir, float& tnear, float& u, float& v)
{
    // TODO: Implement this function that tests whether the triangle
    // that's specified bt v0, v1 and v2 intersects with the ray (whose
    // origin is *orig* and direction is *dir*)
    // Also don't forget to update tnear, u and v.

    Vector3f e1 = v1 - v0;
    Vector3f e2 = v2 - v0;
    Vector3f s = orig - v0;
    Vector3f s1 = crossProduct(dir, e2);
    Vector3f s2 = crossProduct(s, e1);

    float coefficient = dotProduct(s1, e1);


    tnear = dotProduct(s2, e2) / coefficient;
    u = dotProduct(s1, s) / coefficient;
    v = dotProduct(s2, dir) / coefficient;

    if (u > 0 && v > 0 && 1 - u - v > 0 && tnear >= 0) {
        return true;
    }
    return false;
}
```

注意，Moller-Trumbore 算法描述的的是直线和三角形相交情况，而光线是射线

直线要打到三角形内部，并且t不能小于0，因为是光线是射线

![binary](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/5-output.png)
