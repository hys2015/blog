---
title: Bundle Adjustment光束平差法
date: 2017-03-08 21:52:34
tags: 
- Bundle Adjustment
- SLAM
category: 学习记录
---

bundle adjustment（一般翻译为光束平差法，下称光束平差法）定义为一个问题，这个问题需要同时精细化描述场景几何3D坐标、相对运动参数、和相机用来获取图像的光学特性。
<!--more-->

### 定义
给定一个从不同视角刻画一系列3D点的图像，bundle adjustment（一般翻译为光束平差法，下称光束平差法）定义为一个问题，这个问题需要同时精细化描述场景几何3D坐标、相对运动参数、和相机用来获取图像的光学特性。解决这个问题需要参照所有点对应图像的投影的最优性表准来完成。

### 用途

光束平差法几乎是所有基于特征的3D重建算法的最有一步。它相当于一个3D结构和观察参数（比如相机位置、内参矩阵、和径向畸变）的最优化问题，这个问题可以得到对某些观察中固定的噪声的假设下，最好的重建效果。如果图像偏差是**零均值高斯**的（zero-mean Gaussian），那么平差约束就是**最大似然估计**（Maximum Likelihood Estimator）。它的名字表示从3D特征点发出的集束光线，通过最优化调整对应的结构和观察参数，收敛到各个相机的光学中心。

### 通常解法(步骤)
光束平差归结起来就是观察的图片位置和预测的图片点之间的最小化问题，这个问题通常由大量的非线性实值函数的平方和来表示。最小化问题通常使用非线性的最小二乘算法求解。[Levenberg-Marquardt（莱文贝格马奎特）]方法是最成功的方法之一，它易实现而且可以在大范围的初始参数下都可以成功快速收敛。通过在当前估计的邻近区域对函数的线性化迭代，Levenberg-Marquardt算法涉及称为正规方程的线性系统的解法。当求解在束调整的框架中出现的最小化问题时，由于缺乏针对不同3D点和相机的参数之间的交互，正规方程具有稀疏块结构。 这可以被利用以通过使用Levenberg-Marquardt算法的稀疏变体获得巨大的计算益处，其明确地利用正常方程零点模式，避免存储和操作零元素。

### 数学定义
光束平差法相当于联合地精确化初始的相机和结构参数估计集合以找到可以最大精确度预测观察到的图片中的点的位置的参数集合。更正式化一点，假设有$n$个3D点在$m$个视角被观察到，令$x_{ij}$为第$j$幅图片中第$i$个点的投影。令$v_{ij}$为二元值：当点$i$在$j$中可以被看到时，$v_{ij} = 1$，否则为0。同时假设每一个相机$j$使用向量$a_j$进行初始化，每个3D点$i$使用向量$b_i$进行初始化。光束平差法通过调整所有3D点和相机参数来最小化总的重投影(reprojection)的误差，特别地，
$$
\substack{min\\a_j,b_i}\sum^n_{i=1}\sum^m_{j=1} v_{ij} d(\mathbf Q(\mathbf a_j, \mathbf b_i), \mathbf x_{ij})^2
$$
其中$Q(\mathbf a_j, \mathbf b_i)$表示图像$j$上的点$i$预测的投影位置，$d(\mathbf x, \mathbf y)$代表了向量$x$和向量$y$之间的欧式距离。明显的看到，光束平差法通过定义，可以容忍丢失图像投影和最小化物理方面的规范。

-----------------
参考:
**Bundle Adjustment到底是什么？**
https://www.zhihu.com/question/29082659

**Bundle adjustment wiki**
https://en.wikipedia.org/wiki/Bundle_adjustment

**A Generic Sparse Bundle Adjustment C/C++ Package Based on the Levenberg-Marquardt Algorithm**
http://users.ics.forth.gr/~lourakis/sba/
