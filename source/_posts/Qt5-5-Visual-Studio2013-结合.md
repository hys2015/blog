---
title: Qt5.5 + Visual Studio 2013 结合
date: 2016-03-12 14:50:18
tags: 
- Qt
- VisualStudio
- 学习记录
categroy:
- 学习记录

---
Qt Creater 编写代码不够舒服，而vs拥有强大的开发调试环境，在vs上进行qt开发，两全其美。
<!--more-->

# 注意
- 本文适用情况仅限于windows环境下，其他环境不保证具有参考意义。

# VisualStudio
VisualStudio不多介绍，用过的都说好。
到[VS下载页][3]下载vs2013，安装即可，破解什么的网上有好多，实在不会的话就某宝买一个。

# Qt
Qt - 一套代码，创建强大的应用和设备
具体介绍参见[Qt官网][5]
到目前为止（2016年3月），最新版本的Qt是5.5。
安装Qt相当简单，到[下载页面][1]下载好Qt5.5，安装即可。我用的是Qt 5.5.1 for Windows 64-bit (VS 2013, 823 MB)，然后安装就好了。

# 在vs2013上使用qt
在[Qt下载页][1]中选择Other Downloads,下载**qt-vs-addin-1.2.4-opensource**，或者到[这里][4]选择一个网速好点的镜像。
**qt-vs-addin-1.2.5需要将vs2013升级到最新版，否则安装时候会出现问题。**

安装完 qt-vs-addin 之后，在vs的菜单栏上会多出一个菜单选项-qt5，然后，做一下配置。
1. 如图配置qt的路径
![配置Qt路径](http://7xrsid.com1.z0.glb.clouddn.com/qt-vs-qt-options.png)
![填写路径](http://7xrsid.com1.z0.glb.clouddn.com/qt-vs-add-path.png)
这里我的路径是 %你的Qt路径%/Qt/Qt5.5.1/5.5/msvc2013_64 ，然后就配置完毕了。

## Visual Assists x
配置到这里，其实qt已经配置完毕了，但是相信你会看到你编辑器还是会对你的代码报错，毕竟vs再强大，也不可能把所有语法的提示都给你提前做好，所以这时候就需要我们自己安装插件了。
这里我选择的是 Visual Assists X，官网在这里:[Visual Assists X][2]，下载完，安装就好，然后进行一下简单的配置。参考[这里][6]。
然后就可以舒服地写代码了

-END-


[1]: http://www.qt.io/cn/download-open-source/ "Qt Download CN"
[2]: http://www.wholetomato.com/ "Visual Assist x - VAX"
[3]: https://www.visualstudio.com/downloads/download-visual-studio-vs "Download MS VisualStudio"
[4]: http://download.qt.io/official_releases/vsaddin/qt-vs-addin-1.2.4-opensource.exe.mirrorlist "qt vs-add-in 1.2.4"
[5]: http://www.qt.io/cn/ "qt Chinese"
[6]: http://blog.csdn.net/gameloft9/article/details/46403525 "vsx 配置"