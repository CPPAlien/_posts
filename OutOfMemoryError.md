title: Android中的OutOfMemoryError
date: 2016-12-06
tags: [Android, 内存]
categories: Android
---
#### OOM 的产生
在使用C或C++语言时，我们可操作的内存空间就是整个设备的物理内存，程序员需要自己声明内存空间，也需要自己在恰当的时机释放掉内存，一旦出错就会造成内存泄漏。而Java语言为了解决这个问题，在操作系统之上创造了一个Java虚拟机（JVM），让Java语言编译后的字节码运行在此虚拟机之上。启动一个Java应用，会首先启动JVM，JVM 会向操作系统申请所需内存，然后把内存分成为栈内存和堆内存。堆内存用以存放对象实例，并可被Java回收机制回收，一旦剩余堆内存空间不够申请新对象时就会产生OutOfMemoryError异常。

#### Android内存管理
Android的Dalvik虚拟机（DVM）是参考JVM做出来的，所以大同小异。最主要的两个区别是：一，DVM 基于寄存器，而JVM基于栈来进行局部变量的操作，当然在性能上DVM会更快；二，在DVM上运行的是被进一步处理的JAVA字节码，后缀为.dex，.dex 是把Java应用中所有的.class文件合并而成，缩减了包的体积。Android中的 DVM 如 JVM 一样对每个应用可使用的最大内存空间做了限制，每台设备出厂之前厂家就对单个 DVM 实例可使用的最大内存进行了限定。Android届的第一款手机HTC G1的大小为16M。这些信息储存在手机中 `/system/build.prop`配置文件中，如下我这是我使用的华为6p plus手机的相关信息

```
adb shell
shell@hwPE:/ $ cat /system/build.prop
...

dalvik.vm.heapstartsize=8m
dalvik.vm.heapgrowthlimit=192m
dalvik.vm.heapsize=512m
dalvik.vm.heaptargetutilization=0.75
dalvik.vm.heapminfree=512k
dalvik.vm.heapmaxfree=8m
...
```

`dalvik.vm.heapstartsize`为一个应用初始分配的堆大小，越大意味着应用第一次启动时越流畅，但也意味着内存耗用越快。

`dalvik.vm.heapgrowthlimit` 这就是所谓的单个应用可使用的最大内存堆大小。

`dalvik.vm.heapsize` 此项表示应用在manifest中配置android:largeHeap="true"时可使用的最大内存堆大小。

#### 获取内存配置

有些小伙伴可能使用过以下方法来获得内存信息，但可能就和最初的我一样，不知道这些获得的数据到底是啥意思。下面我们就来说说每个方法所获得的数据的意义和特点。

```
Log.e("pengtao", "max memory = " + Runtime.getRuntime().maxMemory());
Log.e("pengtao", "free memory = " + Runtime.getRuntime().freeMemory());
Log.e("pengtao", "total memory = " + Runtime.getRuntime().totalMemory());

ActivityManager am = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
Log.e("pengtao", "memoryClass = " + Integer.toString(am.getMemoryClass()));
Log.e("pengtao", "largememoryClass = " + Integer.toString(am.getLargeMemoryClass()));
```



`Runtime.getRuntime().maxMemory()` 这个参数对应到build.prop中的信息就是在未设置largeHeap为true时会返回`heapgrowthlimit`的大小，而设置了largeHeap为true后，则返回`heapsize`大小。单位为Bytes。

 `getMemoryClass` 所获得的大小不受largeHeap配置影响，永远是`heapgrowthlimit`中大小。而`getLargeMemoryClass`则为`heapsize`大小，两者单位都为M。

最后要想理解`totalMemory`和`freeMemory`概念可以看下下图。



![java runtime memory](https://i.stack.imgur.com/GjuwM.png)



上述日志代码执行后，在我手机上跑出来的结果如下（设置了largeHeap为true），可以把这些数据与`build.prop`中的数据对应起来：

```
12-05 16:01:50.346 29178-29178/? E/pengtao: max memory = 536870912
12-05 16:01:50.346 29178-29178/? E/pengtao: free memory = 8201514
12-05 16:01:50.346 29178-29178/? E/pengtao: total memory = 25361266
12-05 16:01:50.346 29178-29178/? E/pengtao: memoryClass = 192
12-05 16:01:50.346 29178-29178/? E/pengtao: largememoryClass = 512
```

**注：谨慎设置largeHeap，因为越大的堆空间意味着GC（垃圾回收）需要遍历的对象越多，时间就会越久。不过largeHeap配置是从Android 3.0开始支持的，而并发式的GC是从Android 2.3后开始支持，所以虽说GC时间变久了，但不会对应用运行造成很大影响。**



#### Android中OOM

Android应用与Java应用一样，避免OOM就是要剩余足够堆内存供应用使用，要想内存足够呢，首先就需要避免应用存在内存泄漏的情况，内存泄漏后，可使用的内存空间减少，自然就会更容易产生OOM。关于如何避免内存泄漏，可以移步到本人写得另一篇文章[《Android中常见的内存泄漏》](http://www.jianshu.com/p/130d3b22a386)中。还有一个容易产生OOM的情况，就是加载大数据到内存中。要想更深入理解这一点，让我们来做个简单应用，以下为其核心代码：

```
final List<byte[]> container = new ArrayList<>();
findViewById(R.id.get_memory).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Log.e("pengtao", "max memory = " + Runtime.getRuntime().maxMemory());
        Log.e("pengtao", "free memory = " + Runtime.getRuntime().freeMemory());
        Log.e("pengtao", "total memory = " + Runtime.getRuntime().totalMem
        byte[] b = new byte[100 * 1000 * 1000];
        container.add(b);
    }
});
```

同样在这段代码运行在一个堆限制为192M的手机上，当点击两次按钮，应用崩溃，打印的信息如下：

```
12-06 21:17:44.925 4021-4021/? E/pengtao: max memory = 201326592
12-06 21:17:44.925 4021-4021/? E/pengtao: free memory = 16761904
12-06 21:17:44.925 4021-4021/? E/pengtao: total memory = 132736208
12-06 21:17:44.925 4021-4021/? I/art: Starting a blocking GC Alloc
12-06 21:17:44.933 4021-4021/? I/art: Alloc sticky concurrent mark sweep GC freed 124(6KB) AllocSpace objects, 0(0B) LOS objects, 12% free, 110MB/126MB, paused 507us total 7.280ms
... //省略几次GC日志
12-06 21:17:44.988 4021-4021/? W/art: Throwing OutOfMemoryError "Failed to allocate a 100000012 byte allocation with 16777216 free bytes and 81MB until OOM"
```

该应用崩溃的日志打印为：`Throwing OutOfMemoryError "Failed to allocate a 100000012 byte allocation with 16777216 free bytes and 81MB until OOM"`，在崩溃前我们可以从日志中看出系统在努力做了几次GC尝试，但却无法释放足够内存，最终只能跑出OOM异常。异常信息反应出奔溃时的内存状况，我们结合到打印的三个数据，和前面我们所讲内容，正好能对上号。126MB即为total memory，free memory为16M，所以使用了110M空间，因堆限制为192M，堆空闲最少需要预留512K，所以还剩81M可用，而这81M空间无法满足下一次的内存分配，所以产生OOM。



#### 图片处理时

Android编程中，往往最容易出现OOM的地方就是在图片处理的时候，我们先上个数据：一个像素的显示需要4字节（R、G、B、A各占一个字节），所以一个1080x720像素的手机一个满屏幕画面就需要近3M内存，而开发一个轻量应用的安装包大小也差不多就3M左右，所以说图片很占内存。在Android中，图片的资源文件叫做Drawable，存储在硬盘上，不耗内存，但我们并无法对其进行处理，最多只能进行展示。而如果想对该图片资源进行处理，我们需要把这个Drawable解析为Bitmap形式装载入内存中。其中Android的不同版本对Bitmap的存储方式还有所不同。下面是Android官方文档中对此描述的一段话

>On Android 2.3.3 (API level 10) and lower, the backing pixel data for a bitmap is stored in native memory. It is separate from the bitmap itself, which is stored in the Dalvik heap. The pixel data in native memory is not released in a predictable manner, potentially causing an application to briefly exceed its memory limits and crash. As of Android 3.0 (API level 11), the pixel data is stored on the Dalvik heap along with the associated bitmap.

bitmap分成两个部分，一部分为bitmap对象，用以存储此图片的长、宽、透明度等信息；另一部分为bitmap数据，用以存储bitmap的(A)RGB字节数据。在2.3.3及以前版本中bitmap对象和bitmap数据是存储在不同的内存空间上的，bitmap数据部分存储在native内存中，GC无法涉及。所以之前我们需要调用bitmap的recycle方法来显示的告诉系统此处内存可回收，而在3.0版本开始，bitmap的的这两部分都存储在了Dalvik堆中，可以被GC机制统一处理，也就无需用recycle了。

关于bitmap优化，不同版本方法也不想同，2.3.3版本及以前，就要做到及时调用recycle来回收不在使用的bitmap，而3.0开始可以使用`BitmapFactory.Options.inBitmap`这个选项，设置一个可复用的bitmap，这样以后新的bitmap且大小相同的就可以直接使用这块内存，而无需重复申请内存。4.4之后解决了对大小的限制，不同大小也可以复用该块空间。关于inBitmap可参考[官方文档](https://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inBitmap)。

当有多个bitmap需要显示时，可以使用LruCache算法。实践可以参考一个Github开源库：[DaVinci](https://github.com/CPPAlien/DaVinci)

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得易懂且有趣
个人博客：http://pengtao.me, GitHub地址：https://github.com/CPPAlien
