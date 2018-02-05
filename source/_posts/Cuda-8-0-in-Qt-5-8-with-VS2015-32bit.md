---
title: Cuda 8.0 in Qt 5.8 with VS2015 32bit
date: 2017-02-22 15:25:11
tags: cuda qt
category: 学习记录
---

把cuda应用到Qt中没有官方插件也没有完整的教程，网上找到的都是cuda4.0左右版本的教程，我用的是cuda8.0，Qt5.8。自己摸索了一下配置过程。
<!--more-->
## 来自stackoverflow的.pro文件
我的Qt工程结构如下：

    CUDAinQt\
        CUDAinQt.pro
        src\
            main.cpp
            cuda\
                cudaTest.cu

参考stackoverflow的[一篇回答](http://stackoverflow.com/questions/12266264/compiling-cuda-code-in-qt-creator-on-windows)中的.pro文件，如下：
```c++
TARGET = TestCUDA

# Define output directories
DESTDIR = release
OBJECTS_DIR = release/obj
CUDA_OBJECTS_DIR = release/cuda

# Source files
SOURCES += src/main.cpp

# This makes the .cu files appear in your project
OTHER_FILES +=  vectorAddition.cu

# CUDA settings <-- may change depending on your system
CUDA_SOURCES += src/cuda/vectorAddition.cu
CUDA_SDK = "C:/ProgramData/NVIDIA Corporation/NVIDIA GPU Computing SDK 4.2/C"   # Path to cuda SDK install
CUDA_DIR = "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v4.2"            # Path to cuda toolkit install
SYSTEM_NAME = Win32         # Depending on your system either 'Win32', 'x64', or 'Win64'
SYSTEM_TYPE = 32            # '32' or '64', depending on your system
CUDA_ARCH = sm_11           # Type of CUDA architecture, for example 'compute_10', 'compute_11', 'sm_10'
NVCC_OPTIONS = --use_fast_math

# include paths
INCLUDEPATH += $$CUDA_DIR/include \
               $$CUDA_SDK/common/inc/ \
               $$CUDA_SDK/../shared/inc/

# library directories
QMAKE_LIBDIR += $$CUDA_DIR/lib/$$SYSTEM_NAME \
                $$CUDA_SDK/common/lib/$$SYSTEM_NAME \
                $$CUDA_SDK/../shared/lib/$$SYSTEM_NAME
# Add the necessary libraries
LIBS += -lcuda -lcudart

# The following library conflicts with something in Cuda
QMAKE_LFLAGS_RELEASE = /NODEFAULTLIB:msvcrt.lib
QMAKE_LFLAGS_DEBUG   = /NODEFAULTLIB:msvcrtd.lib

# The following makes sure all path names (which often include spaces) are put between quotation marks
CUDA_INC = $$join(INCLUDEPATH,'" -I"','-I"','"')

# Configuration of the Cuda compiler
CONFIG(debug, debug|release) {
    # Debug mode
    cuda_d.input = CUDA_SOURCES
    cuda_d.output = $$CUDA_OBJECTS_DIR/${QMAKE_FILE_BASE}_cuda.o
    cuda_d.commands = $$CUDA_DIR/bin/nvcc.exe -D_DEBUG $$NVCC_OPTIONS $$CUDA_INC $$LIBS --machine $$SYSTEM_TYPE -arch=$$CUDA_ARCH -c -o ${QMAKE_FILE_OUT} ${QMAKE_FILE_NAME}
    cuda_d.dependency_type = TYPE_C
    QMAKE_EXTRA_COMPILERS += cuda_d
}
else {
    # Release mode
    cuda.input = CUDA_SOURCES
    cuda.output = $$CUDA_OBJECTS_DIR/${QMAKE_FILE_BASE}_cuda.o
    cuda.commands = $$CUDA_DIR/bin/nvcc.exe $$NVCC_OPTIONS $$CUDA_INC $$LIBS --machine $$SYSTEM_TYPE -arch=$$CUDA_ARCH -c -o ${QMAKE_FILE_OUT} ${QMAKE_FILE_NAME}
    cuda.dependency_type = TYPE_C
    QMAKE_EXTRA_COMPILERS += cuda
```
可以由`CUDA_SDK`,`CUDA_DIR`看到答主的回答适用于cuda4.2，依照他的设置马上就会发现cuda8.0中没有`CUDA_SDK`对应的文件。google一下发现新版cuda将这两个地方的内容放在了一起，也就是将上述文件中`CUDA_DIR`配置到自己相对应的toolkit目录即可，把所有包含`CUDA_SDK`的行删掉就行了。`CUDA_SDK`主要是跟编译时的lib相关，所以也没有多少行代码跟他相关，删掉就行。

## 找不到*_cuda.o文件

接着，仍然引用那篇回答中将main.cpp和cudaTest.cu源码写好，进行编译。提示错误，说是找不到.cu文件的.obj文件，查看**编译输出**，看来问题出在nvcc的参数上面。可以发现上述4、5、6行定义了目标文件夹，但是前缀都是release，然而我的实验一直都是在debug模式下做的，所以这里应该有问题。

将

    DESTDIR = release
    OBJECTS_DIR = release/obj
    CUDA_OBJECTS_DIR = release/cuda

放在config(...){...}else{...}中，针对debug和release版本，做不同的定义。将前缀分别改为debug和release即可。

## 检测到“RuntimeLibrary”的不匹配项: 值“MDd_DynamicDebug”不匹配值“MTd_StaticDebug”

修改好之后有新的问题，连接错误，提示

    检测到“RuntimeLibrary”的不匹配项:值“MDd_DynamicDebug”不匹配值“MTd_StaticDebug”(在cudaTest_cuda.o中)
    
这是动态编译文件和静态编译文件不匹配导致的，在VS中很容易修改，但是Qt Creator中我还真不知道怎么办。查了查资料有一种办法，在nvcc后面设置参数，将cudart库的编译设置为static就行。

参考： http://stackoverflow.com/questions/26205608/compile-cuda-file-error-runtime-library-mismatch-value-mdd-dynamicdebug-doe


## Value 'sm_10' is not defined for option 'gpu-architecture'
'sm_10'是nvcc中的参数-arch的一个值，新版cuda中这个值已经不支持sm_10了，新的值可以通过nvcc --help命令查看，我这里选用了sm_50。参数的含义在help文档中也有说明。

附上我截取的-arch参数说明。


    Specify the name of the class of NVIDIA 'virtual' GPU architecture for which the CUDA input files must be compiled.
    With the exception as described for the shorthand below, the architecture specified with this option must be a 'virtual' architecture (such as compute_50).
    Normally, this option alone does not trigger assembly of the generated PTX for a 'real' architecture (that is the role of nvcc option '--gpu-code', 
    see below); rather, its purpose is to control preprocessing and compilation of the input to PTX.
    For convenience, in case of simple nvcc compilations, the following shorthand is supported.  If no value for option '--gpu-code' is specified, then the
    value of this option defaults to the value of '--gpu-architecture'.  In this
    situation, as only exception to the description above, the value specified
    for '--gpu-architecture' may be a 'real' architecture (such as a sm_50),
    in which case nvcc uses the specified 'real' architecture and its closest
    'virtual' architecture as effective architecture values.  For example, 'nvcc
    --gpu-architecture=sm_50' is equivalent to 'nvcc --gpu-architecture=compute_50
    --gpu-code=sm_50,compute_50'.
    Allowed values for this option:  'compute_20','compute_30','compute_32',
    'compute_35','compute_37','compute_50','compute_52','compute_53','compute_60',
    'compute_61','compute_62','sm_20','sm_21','sm_30','sm_32','sm_35','sm_37',
    'sm_50','sm_52','sm_53','sm_60','sm_61','sm_62'.

参考：https://devtalk.nvidia.com/default/topic/762051/compile-issues/

## 我的.pro文件

    QT       += core
    
    greaterThan(QT_MAJOR_VERSION, 4): QT += widgets
    
    TARGET = CUDAinQt
    
    SOURCES += src/main.cpp
    
    # This makes the .cu files appear in your project
    OTHER_FILES += cudaTest.cu
    
    # CUDA settings <-- may change depending on your system
    CUDA_SOURCES += src/cuda/cudaTest.cu
    #CUDA_SDK = "C:/ProgramData/NVIDIA Corporation/NVIDIA GPU Computing SDK 4.2/C"   # Path to cuda SDK install
    CUDA_DIR = "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v8.0"            # Path to cuda toolkit install
    SYSTEM_NAME = Win32         # Depending on your system either 'Win32', 'x64', or 'Win64'
    SYSTEM_TYPE = 32            # '32' or '64', depending on your system
    CUDA_ARCH = compute_50           # Type of CUDA architecture, for example 'compute_10', 'compute_11', 'sm_10'
    CUDA_CODE = sm_50
    NVCC_OPTIONS = --use_fast_math
    
    # include paths
    INCLUDEPATH += $$CUDA_DIR/include
    
    # library directories
    QMAKE_LIBDIR += $$CUDA_DIR/lib/$$SYSTEM_NAME
    
    # Add the necessary libraries
    LIBS += -lcuda -lcudart
    
    # The following library conflicts with something in Cuda
    QMAKE_LFLAGS_RELEASE = /NODEFAULTLIB:msvcrt.lib
    QMAKE_LFLAGS_DEBUG   = /NODEFAULTLIB:msvcrtd.lib
    
    # The following makes sure all path names (which often include spaces) are put between quotation marks
    CUDA_INC = $$join(INCLUDEPATH,'" -I"','-I"','"')
    
    # Configuration of the Cuda compiler (NVCC)
    
    # MSVCRT link option (static or dynamic, it must be the same with your Qt SDK link option)
    MSVCRT_LINK_FLAG_DEBUG = "/MDd"
    MSVCRT_LINK_FLAG_RELEASE = "/MD"
    
    CONFIG(debug, debug|release) {
        # Debug mode
        DESTDIR = debug
        OBJECTS_DIR = debug/obj
        CUDA_OBJECTS_DIR = debug/cuda
        cuda_d.input = CUDA_SOURCES
        cuda_d.output = $$CUDA_OBJECTS_DIR/${QMAKE_FILE_BASE}_cuda.o
        cuda_d.commands = $$CUDA_DIR/bin/nvcc.exe -D_DEBUG $$NVCC_OPTIONS $$CUDA_INC $$LIBS \
                          --machine $$SYSTEM_TYPE -arch=$$CUDA_ARCH -code=$$CUDA_CODE \
                          --compile -cudart static -g -DWIN32 -D_MBCS \
                          -Xcompiler "/wd4819,/EHsc,/W3,/nologo,/O2,/Zi" \
                          -Xcompiler $$MSVCRT_LINK_FLAG_RELEASE \
                          -c -o ${QMAKE_FILE_OUT} ${QMAKE_FILE_NAME}
        cuda_d.dependency_type = TYPE_C
        QMAKE_EXTRA_COMPILERS += cuda_d
    }
    else {
        # Release mode
        DESTDIR = release
        OBJECTS_DIR = release/obj
        CUDA_OBJECTS_DIR = release/cuda
        cuda.input = CUDA_SOURCES
        cuda.output = $$CUDA_OBJECTS_DIR/${QMAKE_FILE_BASE}_cuda.o
    #    cuda.commands = $$CUDA_DIR/bin/nvcc.exe $$NVCC_OPTIONS $$CUDA_INC $$LIBS --machine $$SYSTEM_TYPE -arch=$$CUDA_ARCH -code=$$CUDA_CODE -c -o ${QMAKE_FILE_OUT} ${QMAKE_FILE_NAME}
        cuda.commands = $$CUDA_DIR/bin/nvcc.exe $$NVCC_OPTIONS $$CUDA_INC $$LIBS \
                        --machine $$SYSTEM_TYPE -arch=$$CUDA_ARCH -code=$$CUDA_CODE \
                        --compile -cudart static -DWIN32 -D_MBCS \
                        -Xcompiler "/wd4819,/EHsc,/W3,/nologo,/O2,/Zi" \
                        -Xcompiler $$MSVCRT_LINK_FLAG_RELEASE \
                        -c -o ${QMAKE_FILE_OUT} ${QMAKE_FILE_NAME}
        cuda.dependency_type = TYPE_C
        QMAKE_EXTRA_COMPILERS += cuda
    }
