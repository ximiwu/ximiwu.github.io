---
layout:     post
title:      "Games101：光线追踪-作业(3)-路径追踪"
subtitle:   ""
date:       2024-10-13 16:00:00
author:     "西米屋花火"
catalog: true
header-mask: 0.5
tags:
    - Games系列课程
    - Games101
    - graphics
---

在之前的练习中，我们实现了 Whitted-Style Ray Tracing 算法，并![img](file:///C:/Users/admin/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)用 BVH等加速结构对于求交过程进行了加速。在本次实验中，我们将在上一次实验的基础上实现完整的 Path Tracing 算法。至此，我们已经来到了光线追踪版块的最后一节内容。

在本次实验中，你只需要修改这一个函数:

•  **castRay(const Ray ray, int depth)**in Scene.cpp: 在其中实现 Path Trac- ing 算法

可能用到的函数有：

•  **intersect(const Ray ray)**in Scene.cpp: 求一条光线与场景的交点

•  **sampleLight(Intersection pos, float pdf)** in Scene.cpp: 在场景的所有光源上按面积 uniform 地 sample 一个点，并计算该 sample 的概率密度



•  **sample(const Vector3f wi, const Vector3f N)** in Material.cpp: 按照该材质的性质，给定入射方向与法向量，用某种分布采样一个出射方向

•  **pdf(const Vector3f wi, const Vector3f wo, const Vector3f N)** in Ma- terial.cpp: 给定一对入射、出射方向与法向量，计算 sample 方法得到该出射方向的概率密度

•  **eval(const Vector3f wi, const Vector3f wo, const Vector3f N)** in Ma- terial.cpp: 给定一对入射、出射方向与法向量，计算这种情况下的 f_r 值

可能用到的变量有：

•  **RussianRoulette** in Scene.cpp: P_RR, Russian Roulette 的概率



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
    if (t_tmp <= 0) return inter;

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

纠正：上次作业光线与三角形相交忽略了t要大于0

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
        t_enter = std::max(t_enter, t_min);
        t_exit = std::min(t_exit, t_max);
    }
    //在这个作业中，与课上所讲的不同，两个条件都要写成>=，否则部分墙壁会被无视
    //推测是因为：模型厚度太小，产生的包围盒的pMin和pMax几乎相等，如果再加上精度丢失，最后t_enter和t_exit可能就相等了
    return (t_enter <= t_exit) && t_exit >= 0;
}
```

```cpp
Vector3f Scene::castRay(const Ray& ray, int depth) const
{
    //从人眼最初发射的射线，要是没有击中任何物体就什么都不要返回
    Intersection hit_point = intersect(ray);
    //如果击中了，就进入shade递归函数
    if (hit_point.happened) return shade(hit_point, -ray.direction);
    return Vector3f(0, 0, 0);
}
```

```cpp
//相交对象中有坐标、法向量、材质、发光、等信息
//而材质对象提供了BRDF函数、蒙特卡洛积分所需的随机采样函数、PDF函数
Vector3f Scene::shade(Intersection& hit_point, Vector3f wo) const {
    //直接击中发光物体
    if (hit_point.m->hasEmission()) return hit_point.m->getEmission();

    Vector3f L_dir, L_indir;
    //p点是这个shade函数表示的光线击中的第一个点
    Vector3f p = hit_point.coords;
    Vector3f p_normal = hit_point.normal;
    Material& p_material = *(hit_point.m);

    //L_dir
    //从p点直接对光源采样
    Intersection lignt_inter;
    float light_pdf;
    sampleLight(lignt_inter, light_pdf);
    Vector3f p_to_x = lignt_inter.coords - p;
    Vector3f ws = p_to_x.normalized();

    Ray p_to_x_ray(p, ws);
    Intersection p_to_x_inter = intersect(p_to_x_ray);

    //if (std::abs(p_to_x_inter.distance - p_to_x.norm()) < 0.05) {////////////////////////
    //判断p点到光源之间有没有被遮挡
    if (p_to_x_inter.distance - p_to_x.norm() > -0.0005f) {
        L_dir = lignt_inter.emit * p_material.eval(ws, wo, p_normal) *
            std::max(0.0f, dotProduct(ws, p_normal)) * std::max(0.0f, dotProduct(-ws, lignt_inter.normal)) / dotProduct(p_to_x, p_to_x) / light_pdf;
    }

    //ray_die
    //递归停止的条件
    if (get_random_float() > RussianRoulette) return L_dir;

    //L_indir
    //随机采样出一个入射方向
    Vector3f wi = p_material.sample(wo, p_normal).normalized();
    //得到该入射方向的概率
    float p_pdf = p_material.pdf(wo, wi, p_normal);

    //根据文档的提示，pdf太小就应该忽略，否则会有很多噪点
    //推测是因为：pdf无穷小，计算L_indir时除以pdf时就变成无穷大
    if (p_pdf < 0.0005) return L_dir;

    //入射光线
    Ray reflect_ray(p, wi);
    Intersection reflect_point = intersect(reflect_ray);
    //入射光线如果能存在，就一定会击中物体
    //如果击中光源就忽略
    if (reflect_point.happened && !reflect_point.m->hasEmission()) {
        L_indir = shade(reflect_point, -wi) * p_material.eval(wi, wo, p_normal) * std::max(0.0f, dotProduct(wi, p_normal))
            / p_pdf / RussianRoulette;
        return L_dir + L_indir;
    }
    return L_dir;
}
```

![wrong1](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/7-wrong1.png)

bounds的t_enter和t_exit临界判断没有修改的错误结果

![wrong2](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/7-wrong2.png)

光线与三角形相交没有判断t是否大于0的错误结果

![spp1](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/7-spp1.png)

spp=1

![spp16](https://pub-2abc7423feaa4ecb8f59a4cc2d6f2bc5.r2.dev/7-spp16.png)

spp=16
