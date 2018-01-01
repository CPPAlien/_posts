title: 详解Android源码目录
date: 2015-11-10
tags: [Android, 源码]
categories: Android
---
首先推荐大家一个很好的，而且没有被墙的android源码查询网站：http://androidxref.com/

大家可以直接在上面查看源码，而且里面的查询也很方便。
如果是第一次查看源码肯定会被里面的目录名整迷糊了，不知道要在哪个目录下查到自己想要的代码。我这边给大家详细介绍下：

**Bionic** - Google自己开发的安卓C运行库。 在这个文件夹下你可以找到c库的源码，如数学计算和其他的一些核心运行C库源码。（注：一般Linux系统使用glibc, bionic主要以BSD许可形式开源，它仅有200KB是glibc的一半，有更高的效率和低内存占用，更适合移动设备）

**Bootable** - 引导 和 启动相关代码。它对广大设备厂商来说是一个福音，很多设备的boot loaders实行这种fastboot协议，比如Nexus One。

**Build** - 编译系统的实现包含系统所有的核心makefile模板。其中一个很有用的文件 envsetup.sh 可以帮你设定环境变量，编译专有模块和检索一下源码文件。

**Cts** - 兼容性测试。这个测试套确保编译过程符合Android规格。

**Dalvik** - Dalvik 虚拟机的实现源码。

**Development** - 开发相关的一些源码，如sdk、ndk工具。

**Device** - 这里包括硬件模块代码，不同设备，内容也不同。

**External** - 包含所有开源项目的代码，如SQLite, Freetype, Webkit 等。

**Frameworks** - Android框架源码。在这里可以找到Android最核心的实现，比如 包和Activity的管理等。许多的Java与native库映射的api也在这里实现。初期学习源码我们主要关注这个目录下内容。

**Hardware** - 硬件相关源码，如Android硬件抽象层的实现和规范。这个文件夹还包括所涉及的通信模块实现。

**libcore** - 核心java库，大部分内容曲子Apache Harmony的类库子集。（Apache Harmony虚拟机间接催生了Davilk虚拟机）

**libnativehelper** - JNI使用的帮助函数

**Out** - 运行make后生成的一些文件。

**Packages** - 包含系统默认应用的源码，如联系人、日历、浏览器等。

**Prebuilt** - 为方便而提前编译好的二进制文件。

**System** - Android系统核心的源码文件。这是在Dalvik虚拟机和所有java启动前所能运行的最小Linux系统。里面包括init进程、默认的init.rc 脚本 等。

**tools** - 不同IDE工具

为方便大家理解，再贴出一张Android的架构图

![](http://upload-images.jianshu.io/upload_images/1362430-3183afdad473abe0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后再推荐大家一个好用的Chrome插件：[Android SDK Search](https://chrome.google.com/webstore/detail/android-sdk-search/hgcbffeicehlpmgmnhnkjbjoldkfhoin)，安装成功后，打开https://developer.android.com 查询某些View或组件的用法时，可以直接查看它的源码。如下，多了一个 `view source`的按钮。

![](http://upload-images.jianshu.io/upload_images/1362430-821753490fdff964.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参考：
http://stackoverflow.com/questions/9046572/how-to-understand-the-directory-structure-of-android-root-tree
