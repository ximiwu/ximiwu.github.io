---
layout:     post
title:      "Games101：光线追踪-作业(2)-BVH加速"
subtitle:   ""
date:       2024-10-11 21:00:00
author:     "西米屋花火"
catalog: true
header-mask: 0.5
tags:
    - Games系列课程
    - Games101
    - graphics
---

在之前的编程练习中，我们实现了基础的光线追踪算法，具体而言是光线传输、光线与三角形求交。我们采用了这样的方法寻找光线与场景的交点：遍历场景中的所有物体，判断光线是否与它相交。在场景中的物体数量不大时，该做法可以取得良好的结果，但当物体数量增多、模型变得更加复杂，该做法将会变得非常低效。因此，我们需要加速结构来加速求交过程。在本次练习中，我们重点关注物体划分算法 Bounding Volume Hierarchy (BVH)。本练习要求你实现 Ray-Bounding Volume 求交与 BVH 查找。

首先，你需要从上一次编程练习中引用以下函数：

•  **Render()** in Renderer.cpp: 将你的光线生成过程粘贴到此处，并且按照新框架更新相应调用的格式。

•  **Triangle::getIntersection** in Triangle.hpp: 将你的光线-三角形相交函数粘贴到此处，并且按照新框架更新相应相交信息的格式。

在本次编程练习中，你需要实现以下函数：

•  **IntersectP(const Ray& ray, const Vector3f& invDir,**

•  **const std::array<int, 3>& dirIsNeg)** in the Bounds3.hpp: 这个函数的作用是判断包围盒 BoundingBox 与光线是否相交，你需要按照课程介绍的算法实现求交过程。

•  **getIntersection(BVHBuildNode\* node, const Ray ray)**in BVH.cpp: 建立 BVH 之后，我们可以用它加速求交过程。该过程递归进行，你将在其中调用你实现的 Bounds3::IntersectP.

```cpp
inline Intersection Triangle::getIntersection(Ray ray)
{
    Intersection inter;

    if (dotProduct(ray.direction, normal) > 0)
        return inter;
    double u, v, t_tmp = 0;
    Vector3f pvec = crossProduct(ray.direction, e2);
    double det = dotProduct(e1, pvec);
    if (fabs(det) < EPSILON)
        return inter;

    double det_inv = 1. / det;
    Vector3f tvec = ray.origin - v0;
    u = dotProduct(tvec, pvec) * det_inv;
    if (u < 0 || u > 1)
        return inter;
    Vector3f qvec = crossProduct(tvec, e1);
    v = dotProduct(ray.direction, qvec) * det_inv;
    if (v < 0 || u + v > 1)
        return inter;
    t_tmp = dotProduct(e2, qvec) * det_inv;

    // TODO find ray triangle intersection
    inter.happened = true;
    Vector3f offset = t_tmp * ray.direction;
    inter.coords = ray.origin + offset;
    inter.distance = sqrt(dotProduct(offset, offset));
    inter.m = m;
    inter.normal = normal;
    inter.obj = this;
    return inter;
}
```

对比上个作业，返回类型从bool变成了intersection，也就是如果与三角形相交，要在原本基础上返回相交的信息

```cpp
inline bool Bounds3::IntersectP(const Ray& ray, const Vector3f& invDir,
                                const std::array<int, 3>& dirIsNeg) const
{
    float t_enter = -std::numeric_limits<float>::infinity();
    float t_exit = std::numeric_limits<float>::infinity();

    for (int i = 0; i < 3; ++i) {
        float t_min = (pMin[i] - ray.origin[i]) * invDir[i];
        float t_max = (pMax[i] - ray.origin[i]) * invDir[i];

        if (!dirIsNeg[i]) std::swap(t_min, t_max);
        //if (t_min > t_max) std::swap(t_min, t_max);
        t_enter = std::max(t_enter, t_min);
        t_exit = std::min(t_exit, t_max);
    }

    return (t_enter < t_exit) && t_exit > 0;
}
```

这个函数在这个作业是最影响性能的

维护最大值最小值不需要排序，也就是线性复杂度即可，千万不要排序

本人一开始无法理解dirIsNeg有什么用，实际上对于某个维度，由推导出的公式可知如果ray_direction正负确定，则算出来的一组t大小关系就直接被确定不用比大小。所以把正负信息提前存储，空间换时间。

使用dirIsNeg：16s

不使用dirIsNeg，每次都比一次大小：17s

```cpp
Intersection BVHAccel::getIntersection(BVHBuildNode* node, const Ray& ray) const
{
    std::array<int, 3> dirIsNeg;
    dirIsNeg[0] = int(ray.direction.x >= 0);
    dirIsNeg[1] = int(ray.direction.y >= 0);
    dirIsNeg[2] = int(ray.direction.z >= 0);

    Intersection result;
    if (!node->bounds.IntersectP(ray, ray.direction_inv, dirIsNeg)) return result;


    if (node->left == nullptr && node->right == nullptr) {
        return node->object->getIntersection(ray);
    }
    Intersection result1 = getIntersection(node->left, ray);
    Intersection result2 = getIntersection(node->right, ray);
    
    if (result1.distance > result2.distance) return result2;
    else return result1;
}
```

![6-output](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/6-output.png)
