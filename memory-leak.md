title: Android中常见的内存泄漏
date: 2016-04-18
tags: [Android, 内存]
categories: Android
---
#### 写在前面
虽然现在手机的内存不断增大，但Android为了实现不同应用间运行隔离，不至于相互影响，所以对单个应用最大可使用的内存做出了限制。限制大小在不同手机设备和ROM上都可能不一样。如Android界的第一款手机HTC G1是16MB，后来的Nexus One是32MB。所以即使手机内存不断变大，但你开发的应用可使用的内存空间并没有增大很多，这也需要你开发时多注意注意内存问题，遵从最少使用内存的原则，避免内存泄漏的发生，这样不但能让你的应用避免被系统无故杀死，还能让用户使用更加流畅。

如果想查看自己应用可以使用的最大内存空间，可以参考：[《Detect application heap size in Android》](http://stackoverflow.com/questions/2630158/detect-application-heap-size-in-android/2634738#2634738)
如果你实在需要增大自己应用的内存使用大小，可以参考这篇文章：[《How to increase heap size of an android application》](http://stackoverflow.com/questions/11275650/how-to-increase-heap-size-of-an-android-application)

#### 内存泄漏的产生
Android的虚拟机机制模仿JVM，所以也有垃圾回收机制。Android虚拟机中把内存分为两部分，一部分为栈空间，存储一些全局引用和静态变量等值，该空间的分配与回收由系统机制决定，垃圾回收不作用在这块区域；另一部分为堆空间，里面存储是对象的实例，需要开发者主动创建，垃圾回收主要作用在这部分，回收的一个主要策略是检测堆中的对象在栈空间有无对应的引用。如果没有引用指向它，则会被优先回收，如果有引用指向则不会被回收。所以如果开发者没有在适当的时间把一个对象的引用设置为null，则就会可能会产生内存泄漏。在Android中最常见的一个内存泄漏问题就是长时间持有Context。Context在Android中有非常大的作用，比如用来获取资源，所以基本上所有的视图都需要获得Context才能被创建。使用不当则很可能造成内存泄漏。

#### Android中内存泄漏表现
你开发了一个应用，刚开始使用起来还挺流畅，但随着使用时间变长，应用就变得越来越慢，最后导致用户不得重启应用才能继续使用。这就很可能出现了内存泄漏。就像上面提到的，如果说一个静态变量持有了一个Activity的引用，用户打开该Activity，会创建一个Activity的实例，此时即使你关闭该Activity，虽然它不再显示，但它的实例一直会在内存中存在，因为有一个静态变量一直指向它，导致它的内存空间就不会被当做垃圾回收。想想这个Activity中可能包含很多属性，很多视图的信息，它未被释放，会浪费很多内存空间。下面我们从两个个例子入手，讲解下内存泄漏和解决办法。

#### 一个例子
```
private static Drawable sBackground;

@Override
protected void onCreate(Bundle state) {
  super.onCreate(state);
  
  TextView label = new TextView(this);
  label.setText("Leaks are bad");
  
  if (sBackground == null) {
    sBackground = getDrawable(R.drawable.large_bitmap);
  }
  label.setBackgroundDrawable(sBackground);
  
  setContentView(label);
}
```
以上使用一个静态变量来保存一个drawable。从上分析可以看到，一个TextView的局部变量持有了本Activity的引用，因为label是局部变量，所以并不会引起内存泄漏。但紧接着下面，使用了`label.setBackgroundDrawable(sBackground);` 有人可能就会想，这也没啥问题啊，即使`sBackground`作为一个静态变量，持有了一个drawable，这块内存不会被释放，但这块内存毕竟没有持有整个Activity的引用。但实际上你错了。我们来看下View.java中的setBackgroundDrawable源码，源码位置在
（frameworks/base/core/java/android/view/View.java）
```
public void setBackgroundDrawable(Drawable background) {
        ...

        if (background != null) {
            ...

            background.setCallback(this);
            ...
        } else {
            ...
        }

        ...
}
```
其中有一个`background.setCallback(this);`，所以这就导致这个静态变量指向的对象又持有了TextView这个对象的引用。这样，因为是静态变量，像我上一小节所说的，静态变量的生命周期基本和应用同周期，它持有了TextView对象引用，所以TextView不会被回收，然后TextView又持有了整个Activity的引用，所以最后就导致整个Activity在关闭后也不会被系统回收。

当然解决此种问题的方法非常简单，就是把`sBackground `换成非静态变量就行，这样当Activity关闭后，回收机制就能判断，这个Activity的空间不会被使用到了，所以就启动GC。

#### 另一个例子
下面我们再举一个非常常见的例子，Android开发者很喜欢用单例模式，但有些开发者不注意就可能导致内存泄漏，如下：

```
private static DaVinci sDaVinci = null;

public static DaVinci with(Context context) {
    if ( sDaVinci == null ) {
        sDaVinci = new DaVinci(context);
    }
    return sDaVinci;
}
```
大家可能一时觉得这没啥问题啊，但这并不是一个好的写法，因为这可能让用户在使用时把一个Activity的Context传入，导致让一个单例持有了这个Activity的Context引用，造成内存泄漏。一个比较好的写法是使用
`sDaVinci = new DaVinci(context.getApplicationContext());`。因为Application的生命周期本来就是贯穿整个应用的，所以即使被持有也没关系。

#### 几点建议
1，尽量不要用一个生命周期长于Activity的对象来持有Activity的引用。
2，在需要传入Context的时候尽量考虑使用Application的Context，而不是Activity的。
3，在Activity中尽量避免使用生命周期不受控制的非静态类型的内部类，可以使用静态类型的内部类加上弱引用的方式实现。

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得易懂且有趣
个人博客：http://pengtao.me, GitHub地址：https://github.com/CPPAlien
