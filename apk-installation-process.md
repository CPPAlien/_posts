title: APK安装过程
date: 2017-05-15
tags: Android
categories: Android
keywords: APK安装
---
当你安装一个APK包时，有没有思考过此时你的手机做了哪些操作呢？做完这些操作后，一个应用就算在手机上安装成功，进而这个应用就可以被运行呢？下面我们来一步步的探讨下。

当你点击安装后，首先是APK中的`AndroidManifest.xml`被解析，解析的内容会被存储到`/data/system/packages.xml`和`/data/system/packages.list`中。我们打开packages.list和packages.xml，找到Demo应用包名，如下图


![packages.list](http://upload-images.jianshu.io/upload_images/1362430-bae72cbecbb9d4c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![packages.xml](http://upload-images.jianshu.io/upload_images/1362430-41327dd9365a80ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

packages.list中指名了该应用默认存储的位置`/data/data/cn.hadcn.example`，packages.xml中包含了该应用申请的权限、签名和代码所在位置等信息，并且两者都有一个userId为10060。之所以每个应用都有一个userId，是因为Android在系统设计上把每个应用当作Linux系统上的一个用户对待，这样就可以利用已有的Linux上用户管理机制来设计Android应用，比如应用目录，应用权限，应用进程管理等。

做完以上操作，就相当于应用在系统注册了，可以被系统识别。接下来就得保存应用的执行文件了，根据`packages.xml`中指定的`codePath`，创建一个目录，apk会被命名成`base.apk`并拷贝到此，其中lib目录用来存放native库。如下图所示

![](http://upload-images.jianshu.io/upload_images/1362430-cebefbd8b8eac601.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**注：目录是由 包名-1 组成，有时候此处是 -2。这是为了升级使用，升级时会新创建一个-1 或 -2的目录，如果升级成功，则删除原目录并更改packages.xml中codePath到新目录**



此时应用就可以运行了，但如果每次应用运行还得去base.apk中取dex文件，效率就太低了。为了提升效率，Android系统在应用安装时还会做些优化操作，把可所有运行的dex文件单独提取放在一块并做些优化。在Dalvik模式下，会使用dexopt把base.apk中的dex文件优化为odex，存储在`/data/dalvik-cache`中，如果是ART模式，则会使用dex2oat优化成oat文件也存储在该目录下，并且文件名一样，但文件大小会大很多，因为ART模式会在安装时把dex优化为机器码，所以在ART模式下的应用运行更快，但apk安装速度相对Dalvik模式下变慢，并且会占用更多的ROM。

![dalvik-cache](http://upload-images.jianshu.io/upload_images/1362430-3c26477843aecdd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


优化后的dex文件被载入到虚拟机中就可以运行。

参考

http://stackoverflow.com/questions/12442979/android-understanding-the-apk-installation-process

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得易懂且有趣
个人博客：http://pengtao.me, GitHub地址：https://github.com/CPPAlien
