---
title: '[Qt]OpenGL与Qt结合|OpenGL under QML'
date: 2016-03-22 14:24:50
tags: 
- Qt
- 学习记录
category:
- 学习记录
---
写个demo感受一下OpenGL如何在Qt上应用。
<!--more-->
# OpenGL
> 开放图形库（英语：Open Graphics Library，缩写为OpenGL）是个定义了一个跨编程语言、跨平台的应用程序接口（API）的规范，它用于生成二维、三维图像。这个接口由近350个不同的函数调用组成，用来从简单的图形比特绘制复杂的三维景象。而另一种程序接口系统是仅用于Microsoft Windows上的Direct3D。OpenGL常用于CAD、虚拟实境、科学可视化程序和电子游戏开发。
> 
> OpenGL的高效实现（利用了图形加速硬件）存在于Windows，很多UNIX平台和Mac OS。这些实现一般由显示设备厂商提供，而且非常依赖于该厂商提供的硬件。开放源代码库Mesa是一个纯基于软件的图形API，它的代码兼容于OpenGL。但是，由于许可证的原因，它只声称是一个“非常相似”的API。
> -摘自[OpenGL wiki][1]

关于OpenGL的编程展开是个很大的话题，不再赘述，有兴趣的可以参考
- [LearnOpenGL][4]
- [OpenGLTutorial][5]

# OpenGL与Qt结合

我写的demo参考这篇文章：[OpenGLUnderQML][6]
因为之前写文本编辑器的原因，现在看这篇教程感觉容易理解的多。跟着写就好，主要就是理解一下Qt中究竟是如何将OpenGL结合起来的。

这个demo简明扼要，主要思路如下：
1. 写Squircle类和Squirecle类，前者作为组件注册进qml，后者作为Squircle类中的一个成员，处理一些渲染的问题。定义一些信号函数和一些槽函数。
2. 在qml中写好布局
3. 在qml中修改Squircle中的成员t的值
4. 每次渲染都会根据t的值进行，从而画出不同的图案

# 原理
> The OpenGL under QML example shows how an application can make use of the QQuickWindow::beforeRendering() signal to draw custom OpenGL content under a Qt Quick scene. This signal is emitted at the start of every frame, before the scene graph starts its rendering, thus any OpenGL draw calls that are made as a response to this signal, will stack under the Qt Quick items.

> As an alternative, applications that wish to render OpenGL content on top of the Qt Quick scene, can do so by connecting to the QQuickWindow::afterRendering() signal.

> 本例展示了应用程序如何利用`QQuickWindow::beforeRendering`信号在QtQuick场景下画出自定义的OpenGl内容。这个信号会在每一帧渲染之前触发，也就是在图像场景绘制之前，因此任何此信号的槽函数中的OpenGL调用都会栈式堆叠在QtQuick控件之下。
> 另外，如果应用程序想要在QtQuick控件之上绘制图像，那么可以针对`QQuickWindow::afterRendering`信号编写槽函数并与之连接。 


# 总结

## 用到的信号

### QQuickItem::windowChanged

```c++
Squircle::Squircle()
    : m_t(0)
    , m_renderer(0)
{
    connect(this, &QQuickItem::windowChanged, this, &Squircle::handleWindowChanged);
}
```
> void QQuickItem::windowChanged(QQuickWindow *window)
> This signal is emitted when the item's window changes.

### void QQuickWindow::beforeSynchronizing()

> This signal is emitted before the scene graph is synchronized with the QML state.
> This signal can be used to do any preparation required before calls to QQuickItem::updatePaintNode().
> The GL context used for rendering the scene graph will be bound at this point.
> Warning: This signal is emitted from the scene graph rendering thread. If your slot function needs to finish before execution continues, you must make sure that the connection is direct (see Qt::ConnectionType).
> Warning: Make very sure that a signal handler for beforeSynchronizing leaves the GL context in the same state as it was when the signal handler was entered. Failing to do so can result in the scene not rendering properly.

同步的信号

### void QQuickWindow::sceneGraphInvalidated()

> This signal is emitted when the scene graph has been invalidated.
> This signal implies that the opengl rendering context used has been invalidated and all user resources tied to that context should be released.
> The OpenGL context of this window will be bound when this function is called. The only exception is if the native OpenGL has been destroyed outside Qt's control, for instance through EGL_CONTEXT_LOST.
> This signal will be emitted from the scene graph rendering thread.

场景渲染结束，需要释放资源的信号

### void QQuickWindow::beforeRendering()

> This signal is emitted before the scene starts rendering.

> Combined with the modes for clearing the background, this option can be used to paint using raw GL under QML content.

> The GL context used for rendering the scene graph will be bound at this point.

> Warning: This signal is emitted from the scene graph rendering thread. If your slot function needs to finish before execution continues, you must make sure that the connection is direct (see Qt::ConnectionType).

> Warning: Make very sure that a signal handler for beforeRendering leaves the GL context in the same state as it was when the signal handler was entered. Failing to do so can result in the scene not rendering properly.

前面介绍过的图形渲染之前的信号

## 问题

### 文件名的写法 
QOpenGLShaderProgram的方法addShaderFromFile中的第二个参数需要一个文件名，这个文件名我最开始写的是`"vertex.vsh"`，结果一直提示找不到文件，查了查，看到别人代码写成`":/vertex.vsh"`就可以了。不太清楚是为什么。

### 关于Qt main函数中的窗口启动方式
1. 现在见到的有几种，通过ApplicationEngine读取qml，在qml中定义window，然后出现窗口
2. QQuickView直接载入qml文件，然后显示出来

-END-

[1]: https://zh.wikipedia.org/wiki/OpenGL "OpenGL wiki"
[2]: http://baike.baidu.com/item/OpenGL "OpenGL baike"
[3]: https://www.opengl.org/ "OpenGL"
[4]: http://www.learnopengl.com/ "learnopengl"
[5]: http://www.opengl-tutorial.org/ "opengltutorail"
[6]: http://doc.qt.io/qt-5/qtquick-scenegraph-openglunderqml-example.html "openglunderqml"
