---
title: Skinning with Dual Quaternions - 对偶四元数蒙皮
date: 2017-05-16 21:57:36
tags: 
- 算法
- fusion
- rgbd
- 四元数
- 对偶四元数
category: 学习记录
---

蒙皮，就是根据控制单元的变化，控制相应的顶点的变化，线性蒙皮采用线性插值的方法计算变形前后的顶点的位置。

蒙皮的算法主要分为两步：

<!--more-->

一，定义一系列的控制单元，计算几何模型受这些控制单元的影响权重；
二，控制单元改变，几何模型发生响应的形变。

## 基础知识
**Linear Blending Skinning 线性蒙皮**

计算权重是一个很重要的步骤，有一种方法是有解双调和权重。定义控制单元为$H_j \in \Omega, j = 1, ..., m$,每个控制单元$H_j$发生的仿射变换为$T_j$，对于顶点$p \in \Omega$，线性混合蒙皮算法给出变形后的$p'$的位置为控制单元仿射变换$T_j$的加权线性组合：
$$
\mathbf p' = \sum_{j=1}^m w_j(\mathbf p)T_j\mathbf p
$$
$w_j(p)$为顶点$p$受控制单元$H_j$的权重影响。

有界双调和权重$w_j$的计算方法，参考[论文](http://delivery.acm.org/10.1145/2580000/2578850/414ChinaRH.pdf?ip=60.207.237.106&id=2578850&acc=ACTIVE%20SERVICE&key=BF85BBA5741FDC6E%2E478E8F2EC4A762F8%2E4D4702B0C3E38B35%2E4D4702B0C3E38B35&CFID=763502609&CFTOKEN=18159657&__acm__=1494940210_1b518aa5c6721c094dd286d0a7feeef7)第3节“有界双调和权重”。

对权重的$w_j$定义为高阶形状感知平滑度泛函（即拉普拉斯能量）的极小化变量，并服从控制点插值和多个其它理想属性的约束：
$$
argmin_{\substack{w_j, \ j = 1,\dots,m}}\sum^m_{j=1}\frac{1}{2}\int_{\Omega}(\Delta w_j)^2 dV
$$
服从：
$$
\begin{align}
&w_j | _{Hk} = \delta_{jk}, & \tag{1} \\
&w_j | _F为线性, & \forall F \in \mathcal F_c  \tag{2} \\
&\sum_{j=1}^m w_j(p) = 1 & \forall p \in \Omega \tag{3} \\
&0 \le w_j(p) \le 1, j = 1, \dots, m, & \forall p \in \Omega \tag{4}
\end{align}
$$
其中，$\mathcal F_c$是所有控制面的合集，$\delta_{jk}$是[克罗内克函数](http://baike.baidu.com/item/Kronecker%20delta)。
**LBS的塌陷问题**
LBS插值，在控制点旋转导致原本的顶点连接线相交时，插值会导致蒙皮塌陷问题。
![LBS](http://7xrsid.com1.z0.glb.clouddn.com/5-16/lbs.gif "LBS塌陷")

**DQB**
[**Dual Quaternions Linearing Skinning**](https://www.cs.utah.edu/~ladislav/kavan07skinning/kavan07skinning.pdf)
DQB可以解决上述问题。
DQB中使用的是**对偶四元数**。
**常规四元数**的理解见下面链接：
[四元数](https://www.zhihu.com/question/23005815)
[Understanding quaternions](http://www.qiujiawei.com/understanding-quaternions/)
![DQB](http://7xrsid.com1.z0.glb.clouddn.com/5-16/qdb.gif "DQB")

常规四元数只能表示空间的旋转变换，数学形式为$q = [cos(\theta / 2), n_x sin(\theta/2), n_y sin(\theta/2) , n_z sin(\theta/2)]$，其中$(n_x, n_y, n_z)$表示通过原点的旋转轴，$\theta$为旋转角度。

对偶数类似于复数，定义为 $\hat z = a_0 + \varepsilon a_{\varepsilon}, a_0 \in \Bbb R, a_{\varepsilon} \in \Bbb R$，满足$\varepsilon ^2 = 0$，称$a_0$为实部，$a_\varepsilon$为对偶部，$\varepsilon$为对偶算子，类似于复数中的$i$。**对偶四元数**是实部和对偶部都为四元数的对偶数，又可称为八元数。常规四元数只能表示空间旋转，而对偶四元数可以表示空间任意旋转和平移的组合。

对偶四元数定义为：
$$
\hat{\mathbf q} = \hat w + i\hat x + j \hat y + k \hat z
$$
其中，$w$是标量部分（对偶数），$(\hat x, \hat y, \hat z)$是向量部分，$i,j,k$是普通四元数算子。对偶算子$\varepsilon$和四元数算子是可交换的(commute)，比如$\varepsilon i = i \varepsilon$.
所以对偶四元数可以写成两个普通四元数的和的形式：
$$
\hat{\mathbf q} = \mathbf q_0 + \varepsilon \mathbf q_\varepsilon 
$$
其中，$\mathbf q_0, \mathbf  q_\varepsilon$均为普通四元数。

**共轭**
对偶四元数的共轭：
$$
\hat{\mathbf q}^\star = \mathbf q_0^\star + \varepsilon \mathbf q_\varepsilon ^\star
$$

**单位对偶四元数**
$\hat{\mathbf q}$是单位化的，当且仅当$||\mathbf q_0|| = 1, \langle \mathbf q_0, \mathbf q_\varepsilon \rangle = 0$, $||\cdot||$表示模长，$\langle \cdot, \cdot \rangle$表示内积。

**对偶四元数表示3D旋转变换**

当对偶四元数的对偶部$\mathbf q_\varepsilon = \mathbf 0$时，该对偶四元数即表示3D旋转。

对偶四元数可写为
$$
\hat{\mathbf q_r }= [cos(\theta/2), n_xcos(\theta/2), n_ycos(\theta/2), n_zcos(\theta/2)][0 \ 0 \ 0 \ 0] 
$$
其中，对偶部为$\mathbf 0$.

给定向量$\hat v = (v_0, v_1, v_2)$，$q$表示旋转的对偶四元数，那么向量$v$的旋转结果可以写为：
$$
\hat q \hat v \overline{\hat q ^\star}
$$
$\overline{\hat q ^\star}$ 表示共轭对偶四元数。



**对偶四元数表示3D平移变换**
定义一个单位对偶四元数$\hat {\mathbf t} = 1 + \frac{\varepsilon}{2} (t_0 i + t_1 j + t_2 k)$对应为向量$(t_0, t_1, t_2)$表示的平移变换（取变换向量的一半，跟普通四元数表示旋转变换取角度的一半类似）

写成对偶四元数形式：
$$
\hat{\mathbf q_t} = [1 \ 0 \ 0 \ 0 ][0 \ \frac{t_x}{2} \ \frac{t_y}{2} \ \frac{t_z}{2}]
$$

**旋转和平移结合起来**
将上述两种情况结合起来，仍然用$\mathbf q_0$表示旋转对偶四元数，平移变换仍然表示为$1 + \frac{\varepsilon}{2} (t_0 i + t_1 j + t_2 k)$，那么结合起来就得到:
$$
\hat{\mathbf q_t} \hat{\mathbf q_r} = (1 + \frac{\varepsilon}{2} (t_0 i + t_1 j + t_2 k))\mathbf q_0 = \mathbf q_0 + \frac{\varepsilon}{2} (t_0 i + t_1 j + t_2 k)\mathbf q_0
$$

## Dual-quaternion Linear Blending
首先将蒙皮的顶点变换转换称为对偶四元数$\hat{\mathbf q_1},\dots, \hat{\mathbf q_n}$。然后按照权重，计算每个变换对顶点的影响，通过线性组合的方式，写为：
$$
DLB(\mathbf w; \hat{\mathbf q_1, \dots, \hat{\mathbf q_n}}) = \frac{w_1 \hat{\mathbf q_1} + \dots + w_n \hat{\mathbf q_n}}{||w_1 \hat{\mathbf q_1} + \dots + w_n \hat{\mathbf q_n} ||}
$$

## Algorithm

>**输入**： 

>- 对偶四元数$\hat{\mathbf q_1}, \dots, \hat{\mathbf q_n}$(全局变量)
- 顶点位置$v$ 和 法向量$v_n$
- 关节索引 $j_1, \dots, j_n$ 和权重$w_1, \dots, w_n$

>**输出**：变换后的顶点坐标$v'$和法向量$v_n'$

> $\hat{ \mathbf b }= w_1 \hat{\mathbf q_1} + \dots + w_n \hat{\mathbf q_n}$ 
//记$\hat{\mathbf b}$中的实部为$\mathbf b_0$，对偶部为$\mathbf b_\varepsilon$
$\mathbf c_0 = \mathbf b_0 / ||\mathbf b_0||$
$\mathbf c_\varepsilon = \mathbf b_\epsilon / ||\mathbf b_0||$
//记$\mathbf c_0$为 $w_0, x_0, y_0, z_0$
//记$\mathbf c_\varepsilon$为 $w_\varepsilon, x_\varepsilon, y_\varepsilon, z_\varepsilon$
$t_0 = 2(-w_\varepsilon x_0 + x_\varepsilon w_0 - y_\varepsilon z_0 + z_\varepsilon y_0)$
$t_1 = 2(-w_\varepsilon x_0 + x_\varepsilon z_0 + y_\varepsilon w_0 - z_\varepsilon x_0)$
$t_2 = 2(-w_\varepsilon z_0 - x_\varepsilon y_0 + y_\varepsilon x_0 + z_\varepsilon w_0)$
$$
M = 
\begin{pmatrix} 
1 - 2y_0^2 - 2z_0^2 & 2x_0y_0 - 2w_0z_0 & 2x_0z_0 + 2w_0y_0 & t_0 \\
2x_0y_0 + 2w_0z_0 & 1 - 2x_0^2 - 2z_0^2 & 2y_0z_0 - 2w_0x_0 & t_1 \\
2x_0z_0 - 2w_0y_0 & 2y_0z_0 + 2w_0x_0 & 1 - 2x_0^2 - 2y_0^2 & t_2
\end{pmatrix}
$$
$v' = M v$     // $v$的结构为$(v_0, v_1, v_2, 1)$
$v_n' = M v_n$ // $v_n$的结构为$(v_{n, 0}, v_{n, 1}, v_{n,2}, 0)$


有些细节没有说明，可以参考[论文](https://www.cs.utah.edu/~ladislav/kavan07skinning/kavan07skinning.pdf)。



参考：
https://www.zhihu.com/question/32007883
http://www.cnblogs.com/shushen/p/5987280.html
