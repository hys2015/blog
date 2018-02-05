---
title: CUDA 入门
date: 2017-02-18 10:19:48
tags: CUDA, GPU编程
category: 学习记录
---

三维重建离不开大规模的矩阵运算，合理编程利用GPU可以加速这些运算的效率。

<!--more-->

## CUDA教程
参考
http://blog.csdn.net/sunmc1204953974/article/details/51074102
博客中结合例子，把GPU内的基本概念做了比较详尽的介绍，对于入门很友好。

### 让VS IntelliSense识别cuda函数、变量
**针对device代码中的变量，比如threadId.x等**
加入如下#include语句
```
#include <device_launch_parameters.h>
```
**针对device线程同步函数__syncthreads()**
```
#pragma once
#ifdef __INTELLISENSE__
void __syncthreads();
...
#endif
```

### CUDA调试困难？？
linux下使用 cuda gdb
windows下使用 nsight
