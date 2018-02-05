---
title: DynamicFusion - 论文阅读
date: 2017-05-16 16:16:33
tags: 
- 论文阅读 
- fusion
- rgbd
category: 学习记录
---

newcombe大神2015的论文，叼的不行。。。
[论文地址](https://rse-lab.cs.washington.edu/papers/dynamic-fusion-cvpr-2015.pdf)
<!--more-->
## 2. Overview
DF将非刚性形变场景分解到隐性几何表面，重建到刚性规范空间中。还有每帧的形变场，将上述表面转换到当前帧中。
每新的一幅深度帧，以下三种算法依次执行：
1. 建立体元的“模型到帧”变形场参数
2. 将当前深度帧通过变形场映射进规范空间
3. 调整变形场的结构以匹配新加入的几何体

## 3. Technical Details
DNWF方便对场景中的每帧形变进行建模。变形场是所有其他功能的基础。
### 3.1 Dense Non-rigid Warp Field
变形场，提供一个针对每个点的6自由度变换$\mathcal{W}: S \rightarrow SE(3)$
对规范mesh中的每个点$v_c \in S$, 变换$T_{lc} = \mathcal W (v_c)$把规范空间中的点转换到当前空间。

每一帧深度图像，都需要建立变形公式，$\mathcal W_t$，它的表示必须要有效的优化。一种方法是对体素进行密集的取样，但是这样运算量太大，不适用。
**实际上，用一个变换的稀疏集合作为基，用插值的方法就可以定义稠密的体素变形方程。**

使用如下方式，定义变形函数：
**DQB([paper](https://www.cs.utah.edu/~ladislav/kavan06dual/kavan06dual.pdf),[我的博文](http://hengyishu.cn/blog/2017/05/16/Skinning-with-Dual-Quaternions-对偶四元数蒙皮/#more)): dual-quaternion blending双四元数混合**
$$
\mathcal W_t(x_c) = SE3(\mathbf{DQB}(x_c)).
$$
其中
$$
\mathbf{DQB}(x_c) = \frac{\sum_{k \in N(x_c)}\mathbf w_k(x_c) \mathbf {\hat q_{kc}}}{||\sum_{k \in N(x_c)} \mathbf w_k(x_c) \hat q_{kc}||}
$$
每个单位的双四元数$\mathbf{\hat q_{kc} \in \Bbb R ^8}$。
$N(x)$是点$x$的$k$最近邻的变换结点，$\mathbf w_k$定义了权重来调整每个结点的影响半径。
$\mathbf{SE3}(·)$将四元数转变回$\mathbf{SE(3)}$变换矩阵。

$t$时刻的变形场的状态为$\mathcal W_t$，定义为$n$个变换结点集合的值$\mathcal N^t_{\mathbf{warp}} = \{dg_v, dg_w, dg_{se3}\}_t$。对结点$i = 1 \dots n$，都在标准空间中有个位置$dg_v^i \in \Bbb R^3 $，相关的变换$T_{ic} = dg^i_{se3}$，径向基权重$dg_w$控制变换的程度 $w_i(x_c) = \mathrm{exp}(-||\mathbf{dg}_v^i - x_c||^2 / (2(\mathbf{dg}_w^i)^2))$.

每个半径参数$dg^i_w$设定为保证结点的影响能够覆盖到相邻的若干结点，取决于结点取样的系数程度，$ \S 3.4 $详细介绍。

变形场对所有支持的空间都有效，所以可以对位置和朝向（方向向量）都有效。而且，空间的尺度变化也能够通过相邻点的收敛和发散来表示空间的收缩和扩大。最后，可以用因子表示对体素中所有的点都有效的变换。引入将模型变换至当前帧的变换，$T_{lw}$，并写入体素变形函数，得到:
$$
\mathcal W_t(x_c) = T_{lw} SE3(\mathbf{DQB}(x_c)).
$$

### 3.2 Dense Non-rigid Surface Fusion
此节介绍根据变形场$\mathcal W_t$，更新标准模型几何形状。标准空间(canonical space)$S$，通过取样的$TSDF \ \  \mathcal V : \mathsf S \mapsto \Bbb R^2$，体素域$\mathsf S \subset \Bbb N^3$。取样函数保有每个对应取样点$x_c$的体元$x\in \mathsf S$，二元组$\mathcal V(x) \mapsto [v(x) \in \Bbb R, w(x) \in \Bbb R ]^\top$保有了点$v(x)$所有投影$TSDF$目前的加权平均值，$w(x)$为相关的权重之和。

本文使用跟KinectFusion的方法操作非刚体变形场景。给定深度图$D_t$，通过建立的变换，将体素中心$x_c \in S$变换到当前帧$(x^\top_t, 1)^\top = \mathcal W(x_c)(x_c^\top, 1)^\top$，通过把变换后的中心投影到深度图来维持TSDF表面融合操作。这就可以让TSDF中的体素通过在变形帧计算投影TSDF更新，而不用必须在当前帧中的变形TSDF进行再采样。变形过的标准点（canonical point）投影的符号距离(signed distance)是:
$$
psdf(x_c)  =[K^{-1} D_t(u_c) [u_c^\top, 1 ]^\top]_z -[x_t]_z
$$
$u_c = \pi(Kx_t)$是体素中心投影的像素点。$[·]_z$表示在帧空间的光$(z)$轴上的距离。K是内参，$\pi$是投影变换。对每个体素$x$, 用TSDF融合，更新TSDF，以便于它包含到变形帧投影的SDF：
$$
\mathcal V(x)_t = 
\begin{cases}
[v'(x), w'(x)]^\top, &if \ \ \  psdf(dc(x)) > -\tau \\
\mathcal V(x)_{t-1}, &otherwise
\end{cases}
$$
$dc(.)$将离散体素点转换到连续的TSDF域中。
$\tau > 0$是截断距离，根据加权平均策略，更新TSDF，加权截断方式：
$$
\begin{align}
v'(x) &= \frac{v(x)_{t-1}w(x)_{t-1} + min(\rho, \tau)w(x)}{w(x)_{t-1} + w(x)} \tag{1} \\
\rho &= psdf(dc(x)) \tag{2} \\
w'(x) &= min(w(x)_{t-1} + w(x), w_{max}) \tag{3}
\end{align}
$$
权重$w(x)$表示在深度帧投影点观察的深度的不确定性，也表示与$x_c$处变形函数的关联程度的不确定性。非刚性变形的情况下，点$x_c$离已经映射好和观察到的区域，就越不确定它的变换。

**使用$x_c$到它k最近临的形变结点的平均距离作为上述不确定性和尺度的代理：**$w(x) \propto \frac{1}{k} \sum_{i \in N(x_c)} ||dg_w^i - x_c||_2$. 用每个体素的变换将相关的体素变换到当前非刚性帧。

### 3.3 Estimating the Warp-field State
文章基于当前深度图$D_t$和当前重建$\mathcal V$，通过可以被参数最小化的能量函数，构建变形场$\mathcal W_t$中的变换$dg_{se3}$，能量函数为:
$$
E(\mathcal W_t, \mathcal V, D_t) = Data(\mathcal W_t, \mathcal V, D_t) + \lambda Reg(\mathcal W_t, \mathcal E)
$$
$Data(\mathcal W_t, \mathcal V, D_t)$为模型到帧ICP的花费，规格化项$Reg(\mathcal W_t, \mathcal E)$防止不平滑的移动场，并且保证被**边集合**$\mathcal E$连接的点进行ARAP变换。$\lambda$是平衡系数。
#### 3.3.1 Dense Non-Rigid ICP Data-term
为了构建所有的变换矩阵$T_{ic}$和$T_{lw}$，文章将标准体元中零面（zero level set)提取出来，变换到当前帧中。

**surface prediction and data-association**
TSDF$\mathcal V$中的零面，通过marching cubes方法提取出来，并且以顶点-法向对的方式存储到标准空间：$\hat{ \mathcal V} = {V_c, N_c}$，通过变形场$W_t$将$\hat{\mathcal V_c}$变换到$\hat{\mathcal V_w}$.

通过将变换后的平面$V_w$投影到当前帧，渲染出标准空间中的点，来得到（最终）模型的几何和当前帧的数据关联（对应）。
结果可以得到标准模型中在当前帧可见的几何面的预测:$\mathcal P(\hat{\mathcal V_c})$.将这个预测保存为图像对$\{v, n\} : \Omega \mapsto \mathcal P(\hat{\mathcal V_c})$,$\Omega$是预测的图像，保存了渲染的标准空间的顶点和法向。

给出当前时间帧优化过的变换参数，预测可见的平面应该会向当前阵的表面变换的更近，噪音更少。当前表面$\mathbf{vl}: \Omega \mapsto \Bbb R^3$，由深度图反投影表示$[\mathbf{vl}(u)^\top, 1]^\top = K^{-1}D_t(u)[u^\top, 1]^\top$.

每像素的稠密模型到帧的点到面误差可以量化上述变化，文章通过Tukey惩罚函数$\psi_{data}$，整合了整个图片域$\Omega$:
$$
Data(\mathcal W, \mathcal V, D_t) = \sum_{u\in \Omega} \psi_{data}(\hat n_u^\top(\hat v_u - vl_{\tilde u}))
$$

将模型顶点$v(u)$变换到当前帧的预测位置: $\tilde T^u = \mathcal W(v(u))$，得到响应的顶点-法向预测值： $\hat v_u = \tilde T^u v(u), \hat n_u = \tilde T^u n(u)$，模型的顶点-法向数据和当前帧顶点-法向数据关联可以通过透视投影像素：$\tilde u = \pi (K \hat v_u)$

每个数据项的加数仅依赖计算变形场$\mathcal W$的n的变换的子集。

#### 3.3.2 Warp-field Regularization

非刚体TSDF融合不仅需要估算可见面的变形，要估计整个正规空间$\mathsf S$。这样才能够对即将进入场景的面进行建模。但是当前不可见的面不会有关联数据项。文章认为不可见面的运动是分段平滑的。

规则化项
$$
Reg(\mathcal W, \mathcal E) = \sum^n_{i=0} \sum_{j\in \mathcal E(i)} \alpha_{ij} \psi_{reg} (T_{ic}dg_v^j - T_{jc} dg^j_v)
$$
$\mathcal E$定义了规格化图拓扑，$\alpha_{ij}$定义了边关联的权重，设定为$\alpha_{ij} = max(dg^i_w, dg^j_w)$

**Hierarchical Deformation Tree**
构建层次变形树可以提高变形场的稳定性和减小能量函数优化的开销。

给定当前形变结点集合$\mathcal N_{warp}$，构建一个规格化结点的层次结构$\mathcal N_{reg} = \{r_v, r_{se3}, r_w\}$，3.4节介绍如何构建。

文章不使用变形函数$\mathcal W$中的$\mathcal N_{reg}$，它被用来引入跨越变形函数的长范围（long range）的规格化，这些变形函数已经经过优化，降低了计算复杂度。

规格化图拓扑可以简单的通过把层中（从$\mathcal N_{warp}$开始）的每个点与它下一层的k临近点添加边来构建。

整个融合过程中需要以帧率的速度更新变形结点和规格化图。

#### 3.3.3 Efficient Optimization

将总能量$E$最小化，可以估计出所有的变换参数$T_{lw}, dg_{se3}, r_{se3}$.
用[高斯牛顿法](http://www.voidcn.com/blog/jinshengtao/article/p-6004352.html)求解最小化

最小化能量公式中的复杂度集中在近似化海因公式：$J^\top J = J^\top_d J_d + \lambda J_r^\top J_r$.

### 3.4 Extending the Warp-field
这里扩展了变形场的参数，确保所有的变形都被表示出来，不管是新加入场景的平面还是即将被观察到的空间。包括，增量更新图节点$\mathcal N_{warp}$，然后重新计算层次边拓扑图$\mathcal E$以扩展规格化数据，来容纳新的结点。

**Inserting New Deformation Nodes into $\mathcal N_{warp}$**
在完成非刚体的TSDF的融合之后，可以提取出标准空间中的估计平面为线模型$\mathcal V_c$，给定当前结点们$\mathcal N_{warp}$，可以计算出每个形变函数可以覆盖到提取平面的程度。这需要计算每个顶点到它们的支持结点的向量距离。当距离满足以下条件时，即为不得到支持的顶点：$min_{k\in N(x_c)}(\frac{||dg_v^k-v_c||}{dg_w^k}) \ge 1$. 所有不受支持的点，都会被用半径搜索平均（radius search averaging）的方法进行稀疏降采样，收缩这顶点到新的位置$\tilde{dg_v}$的集合中，这些位置之间至少相隔$\epsilon$的距离。$\epsilon$定义了运动场(motion field)的有效分辨率。每个新的结点中心 $dg ^\star _v {\in} \tilde{dg_v}$ 需要当前变换的初始值，通过当前变形的**DQB**直接得到 $dg_{se3} ^\star \leftarrow \mathcal W_t(dg_v^\star)$ .最后，更新当前变形结点的集合，对应当前的时间$\mathcal N^t_{warp} = \mathcal N_{warp}^{t-1}\cup\{\tilde{dg_v}, \tilde{dg_{se3}}， \tilde{dg_w}\}$. 每次结点更新，也会更新预先计算的$k$临近点域$\mathcal I$.

**Updating the regularisation Graph $\mathcal E$**
给定新跟新的变形结点的集合，构建一个$L \ge 1$层级的规格化图结点层次结构，$l = 0$层是$\mathcal N_{warp}$。计算$l = 1$层是通过在变形场结点$dg_v$上半径搜索再采样到增长抽取的半径$\epsilon \beta ^l, \beta gt 1$，然后通过在当前更新过的$\mathcal W_t$使用**DQB**计算初始结点变换。

接着构建全新的规格化边集$\mathcal E$，从$l = 0$的边开始，到$l = 1$层中的$\mathcal N_{reg}$. 细粒度层($l = 0$)的每个结点都要添加一条到它在粗粒度层$k$临近对应结点的边。



