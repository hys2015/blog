---
title: k-d树
date: 2017-02-16 16:44:43
tags: 数据结构
category: 学习记录
---
k-d树是查找相邻点很重要的一种数据结构。
<!-- more -->

> 索引结构中相似性查询有两种基本的方式：一种是**范围查询**（range searches），另一种是**K近邻查询**（K-neighbor searches）。
范围查询就是给定查询点和查询距离的阈值，从数据集中找出所有与查询点距离小于阈值的数据；
K近邻查询是给定查询点及正整数K，从数据集中找到距离查询点最近的K个数据，当K=1时，就是最近邻查询（nearest neighbor searches）。

特征匹配算子可以分为两类，一类是线性扫描法（穷举），一类是建立数据索引，然后进行快速匹配。

k-d树是索引树的一种。划分的空间没有混叠（clipping）的，另一种划分空间是有混叠（overlapping）的。

k-d树的算法分为两类，一类是有关k-d树的构建算法，一类是在k-d树上进行邻近查找的算法。


### k-d树的构建算法
k-d是k-dimension的缩写，是对数据点在k维空间中划分的一种数据结构。k-d树实际上是一种二叉树。
它结点内容包括：

|数据名 | 类型 | 解释 |
|------| ----- | ---- |
|dom_elt| k维的向量 | k维空间中的一个点，k-d树在某个维度以这个点为界进行划分 |
|split| 正数 | 表示进行划分的维数的序号 |
|left| kd-tree| 左子树|
|right|kd-tree| 右子树|

注意split是当前结点进行划分的维数的序号，序号只是为了在算法中表示方便，其实就是维度的标识。

**算法描述**

**split选取**
选取split时，为了得到较好的划分效果（较高的分辨率），通常做法是，计算每个维度的方差$\sigma_i$，如果$\sigma_p$是$\sigma_i$中最大的，那么取维度$p$作为split。

**dom_elt选取**
将所有元素按照split维的数值大小进行排序，取最中间的那个元素作为dom_elt。

**建立子树**
假设ele为k维空间中的任意，对dom_elt一侧的数据($ele[split]\le dom\_elt[split]$)建立左子树，树根为当前结点的左子结点。另一侧的数据($ele[split] > dom\_elt[split]$)建立右子树，树根为当前结点的右子节点。

> 伪代码
算法：构建k-d树(build_kd_tree)
输入：k维点集ele_set
输出：k-d树 kd_t

> 1. 如果ele_set 为空，则返回空
> 2. 调用选择分裂点的子程序，得到分裂点dom_elt和分裂的维度split
> 3. ele_set_left = $\{e | e \in ele\_set, e[split] \le dom\_elt[split] \} $ 
ele_set_right = $\{e | e \in ele\_set, e[split] \gt dom\_ elt[split]\} $
>4. build_kd_tree(ele_set_left)
build_kd_tree(ele_set_right)


分裂点指的是结点中的dom_elt值，标识了当前分裂的界线。
### k-d树的邻近查找

```c++
算法：k-d树查找target点的最近邻点(find_kd_tree)
输入：k-d树kd_tree, 目标点 target
输出：目标的最近邻点nearest, 最近距离dist

1. 如果kd_tree为空，则设dist为无穷大，返回
2. 搜索k-d树直到叶子结点，记录搜索过的路径
pSearch 指向 kd_tree的根节点
while(pSearch指向的结点p不是叶子结点)
{
    将p加入到search_path中
    if(target[p.split] <= p.dom_elt[p.split]){
        pSearch指向当前结点的左子树    
    }else{
        pSearch指向当前结点的右子树
    }
}
取出search_path中最后一个赋值给nearest
dist = Distance(nearest, target)
3. 回溯搜索路径
while(search_path 不为空)
{
    pBack = search_path最后一个元素
    if(pBack 为叶子结点)
    {
        if(Distance(nearset, target) >
           Distance(pBack->dom_elt, target) )
        {
            nearset = pBack->dom_elt
            dist = Distance(pBack->dom_elt, target)
        }
    }
    else
    {
        s = pBack->split
        if(abs(pBack->dom_elt[s] - target[s]) < dist)
        {
            if( Distance(nearest, target) > 
                Distance(pBack->dom_elt, target) )  
            {  
                nearest = pBack->dom_elt;  
                dist = Distance(pBack->dom_elt, target);  
            }  
            if(target[s] <= pBack->dom_elt[s]) 
                pSearch = pBack->right;  
            else  
                pSearch = pBack->left;
            if(pSearch != NULL)  
                pSearch加入到search_path中      
            } 
        }
    }
}
```

**参考:**
http://underthehood.blog.51cto.com/2531780/687160
[k-d tree wikipedia](https://en.wikipedia.org/wiki/K-d_tree)
http://www.cnblogs.com/eyeszjwang/articles/2429382.html