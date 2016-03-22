---
title: '[Qt]使用C++编写QML组件'
date: 2016-03-18 21:58:00
tags:
- 学习记录
- Qt
category:
- 学习记录
---

初学Qt，官网文档看的有点乱乱的，经常找不到自己需要的东西。于是决定跟着官网的一篇文章写一个Demo--一个简单的文本编辑器，参考教程在[这里][3]
<!--more-->
# QML & QML modules
QML (Qt Markup Language[2] 或 Qt Meta Language 或 Qt Modeling Language[3]) 是基于JavaScript、宣告式编程的编程语言，用于设计用户界面为主的应用程序。它是Qt Quick，诺基亚开发的用户界面创建包的一部分。QML 主要用于移动应用程序，注重于触控输入、流畅的动画（60张/秒）和用户体验。QML documents 描述元素的对象树。

QML 元素可以通过标准 JavaScript 增强，包括这 inline 和引入.js 档。元素可以也无缝集成和使用 Qt 框架的 C++ 组件扩展。

语言的名称是 QML。runtime的名称是 QQuickView。

===以上来自[QML wiki][2]===

简而言之，QML是一个基于javascript的语言，能够方便地进行用户界面设计，也能够通过引入js文件和访问c++对象来拓展功能，使用方便又功能强大。

QML官方参考文档: [QtQML][1]

# QML访问C++对象
使用Qt的重点就在于如何将QML与自己需要实现的功能代码相结合，既满足QML简单的特点，又能实现自己所需要的功能。
开头就提到，我是跟着这篇教程--[Getting Started Programming with Qt Quick][3]做的Demo，想要先熟悉一下开发过程。
接下来就是入坑爬坑的阶段了。

环境说明： 
- win10
- Qt5.5 for msvs2013 64bit
- QtCreator

**教程内的源码包含在Qt的Example文件夹内！地址类似这样 Qt\Qt5.5.1\Examples\Qt-5.5\quick\tutorials\gettingStartedQml**

## 新建工程
想了解Qt Quick 1 vs Qt Quick 2的同学请移步: [QML1-vs-QML2][4]
这个过程教程里没有详细说明，因为使用的控件都是自己写的，所以新建一个Qt Quick的工程就好。

## 写qml控件
跟着教程写控件包括 Button, EditMenu, FileMenu，MenuBar这些东西，参考源码的内容写就好，这时候因为有些交互代码（比如点击按钮要进行保存之类的）没有相应的C++代码支持，所以会报错，这些报错的内容可以先不写，后面再补上。

## 写C++对象
也是参考源码来写，没什么大问题，这个地方不会有什么报错的，就跟着Qt的教程把需要的写的宏和语句写好了就行

### 重要的宏
重要的宏有这几个
```
	//类定义的时候需要
	Q_OBJECT
	
	//设定属性已经读写属性对应的C++函数以及发出的那种信号
	Q_PROPERTY(type propertyName READ propertyName Write setPropertyName NOTIFY signalName)
	
	//可以被QML调用的函数
	Q_INVOKABLE void yourFunctionName( args.. );
	
```
为什么要用这些宏，参考官方文档：
[Q_OBJECT][5]
[Q_PROPERTY][6]
[Q_INVOKABLE][7]
更多宏，请参考[QObject][8]的Macros小节

## QML访问C++对象
这是个难点，把QML的样式写好之后，需要添加功能，相应的C++代码也写好了，但是要结合起来还是有点麻烦。
上面提到的教程是通过qmlscene带参运行，来达到qml和C++代码的结合，但是这样的话就不能在Qt Creator中写代码时，看到提示了，很难受，所以就得看看其他的办法。

### Qt Creator 中使用qmlscene运行项目
顺带说一下如何在Qt Creator中使用qmlscene运行demo中的源码
1. 使用qmake生成imports文件夹，生成文件夹的地址得在 `QtCreator -》 项目` 中构建设置下的shadow build是否打钩，如果打钩，那么生成的文件会在 构建目录中。没有打钩的话，就是在项目文件夹下生成。
2. 把imports放到demo源码的目录
3. 配置运行参数
在 QtCreator -》项目 中的设置如图
![qtCreator-qmlscene](http://7xrsid.com1.z0.glb.clouddn.com/qt-qml-qmlscene.png)
参数也如图中所示，接下来运行即可
4. 注意：第一步中生成imports的文件夹要和第3步中的运行时的版本是一致的，也就是说如果生成imports时是release版，那么第3部构建时也需要是release，如果是debug版的就都需要时debug版，不能混用。

### 不使用qmlscene运行
qt自带的源码中的项目打开，目录结构是这样的
![目录结构](http://7xrsid.com1.z0.glb.clouddn.com/qt-qml-getstartedqml.png)
可以看到，没有qml相关的文件
我也不太明白为什么要这样设置，但是使用上面提到的qmlscene的方法确实可以运行。
然后我就想了想办法，把c++代码和qml结合起来。

在我们自己建的项目中，把写好的c++代码的头文件和源代码文件添加进项目，然后在main函数QmlApplicationEngine初始化之前把我们自己写的FileDialog组件注册一下就好，代码如下(添加的就是星号之间的代码)
```c++
//main.cpp
#include <QGuiApplication>
#include <QQmlApplicationEngine>

#include "FileDialog/dialogPlugin.h"

int main(int argc, char *argv[])
{
    QGuiApplication app(argc, argv);
	
//****************************
    DialogPlugin plugin;
    plugin.registerTypes("FileDialog");
//****************************

    QQmlApplicationEngine engine;
    engine.load(QUrl(QStringLiteral("qrc:/main.qml")));

    return app.exec();
}

```

项目结构如下
![motepad](http://7xrsid.com1.z0.glb.clouddn.com/qt-qml-motepad.png)

然后再进行构建，运行就可以运行了。

而且，这样写的话，在qml文件中import FileDialog之后，可以看到自己定义的那些属性和可以被引用的方法！

## 踩坑记录

### 必要的qt头文件没有include
必要的qt头文件没有include，导致构建时出现了莫名其妙的错误，看log也看不出来代码哪里写错了，全是stdio.h中的错误。以后要谨记这点。


# 参考文档

- [Writing QML Extensions with C++][9]


[1]: http://doc.qt.io/qt-5/qtqml-index.html "Qt QML"
[2]: https://zh.wikipedia.org/wiki/QML "QML wikipedia"
[3]: http://doc.qt.io/qt-5/gettingstartedqml.html "Getting Started Programming with Qt Quick"
[4]: https://wiki.qt.io/QML1-vs-QML2 "QML1-vs-QML2"
[5]: http://doc.qt.io/qt-5/qobject.html#Q_OBJECT "Q_OBJECT"
[6]: http://doc.qt.io/qt-5/qobject.html#Q_PROPERTY "Q_PROPERTY"
[7]: http://doc.qt.io/qt-5/qobject.html#Q_INVOKABLE "Q_INVOKABLE"
[8]: http://doc.qt.io/qt-5/qobject.html "QObject"
[9]: http://doc.qt.io/qt-5/qtqml-tutorials-extending-qml-example.html "Writing QML Extensions with C++"