---
title: 迭代最近点方法ICP
date: 2017-02-16 16:57:57
tags: ICP, 点云对齐
category: 学习记录
---

ICP迭代最近点方法，是三维建模中将点云模型对齐的一种常用方法。

<!--more-->
## ICP算法

[wiki](https://en.wikipedia.org/wiki/Iterative_closest_point)

>ICP算法中，一幅点云，称为**参考**或者**目标点云**，是固定不动的。另一幅，称为**源点云**，被算法变换到与参考点云最佳匹配的位置。算法迭代的得出变换矩阵（平移和旋转的组合），这个矩阵需要最小化从源到目标点云的距离。

>**输入：** 目标点云和源点云；两幅点云对齐的评判标准（非必须）；终止迭代的条件
**输出：**计算的变换矩阵

>**主要步骤：**

> 1. 对源点云中的每一个点，找到目标点云中的最近点，作为匹配位置
2. 使用均方差函数估算上述步骤找到的、可以将源点云变换到上一步找到的匹配位置的刚性变换。
3. 对源点云应用上述刚性变换
4. 迭代



[数学解释与稳定性分析](http://101.96.8.164/www.cs.princeton.edu/~smr/papers/icpstability.pdf)

> $$
E = \Sigma_i [(p_i - q_i) \cdot 
n_i + t \cdot n_i +\\
\alpha(p_{i,y}n_{i,z} - p_{i,z}n_{i,y}) + 
\\ \beta (p_{i,z}n_{i,x} - p_{i,x}n_{i,z}) + \\ \gamma(p_{i,x}n_{i,y} - p_{i,y}n_{i,x})] ^ 2
$$
设 $c = p \times n$ 且
$$
r = 
\begin{pmatrix}
\alpha \\ \beta \\ \gamma
\end{pmatrix}
$$
则E可以写为
$$
E = \Sigma_i[(p_i - q_i)\cdot n_i + t \cdot n_i + r \cdot c_i] ^ 2
$$

这里依据的是向量积的坐标表示。
https://zh.wikipedia.org/wiki/%E5%90%91%E9%87%8F%E7%A7%AF
wiki中“三维坐标”一节中的说明。

**实现参考**
http://www.cnblogs.com/sddai/p/6129437.html