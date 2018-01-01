title: ImageView中的ScaleType详解
date: 2015-12-30
tags: [Android, 布局]
categories: Android
---
#### 官方介绍
![官方文档上的介绍](http://upload-images.jianshu.io/upload_images/1362430-631d849ab9fe73de?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下面举个例子对以上几点属性逐条说明

#### 准备
准备一张400x300的图片，命名为：test_400x300，写一个简单的布局，观察右边preview预览图的变化。
![试验环境](http://upload-images.jianshu.io/upload_images/1362430-ea0f65b2f90fa71a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从图中我们可以看到，此控件的长宽布局都用in单位，之所以长宽都用in单位，是为了框定一个不受屏幕分辨率的区域。这样一张400x300px的图片放到不同的drawable目录下后，在屏幕上就会占用不同的英寸大小，此时scaleType不同而带来的变化更容易观察。
此方面知识如果还不清楚可以参考本人的另一篇博文：[《Android布局中的尺寸单位介绍》](http://www.jianshu.com/p/0296fada6df3)

#### FIT_CENTER
fitCenter是ImageView控件的**默认ScaleType**。
它表示把一个图片缩放到当前View大小，小于的图片会放大，大图会缩小。事例图片见上文”准备“中的图。

#### CENTER_INSIDE
把图片缩小到ImageView区域中，并居中显示。它与FIT_CENTER的区别在于，如果是小于该控件的图，则不会放大，而是维持图的大小直接居中显示。
![centerInside](http://upload-images.jianshu.io/upload_images/1362430-3eb4f09368400bbd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 
android:scaleType="centerInside"
test_400x300.png在drawable-hdpi中

#### CENTER
直接把图片居中显示，不进行任何缩放动作，在控件区域内的则显示，不在就不显示。当图片小于控件时，与CENTER_INSIDE作用一样。

![center](http://upload-images.jianshu.io/upload_images/1362430-e24a61d0afc9e27d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
android:scaleType="center"
test_400x300.png在drawable-mdpi中
**注：因为mdpi是指160dpi，所以用400/160大于2in，所以宽度多出的部分就没有显示出来**

#### CENTER_CROP
放大或缩小图片直到图片的中间区域恰好可以把控件区域填满。
![centerCrop](http://upload-images.jianshu.io/upload_images/1362430-ab3dc10afba338b0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
android:scaleType="centerCrop"
test_400x300.png在drawable-mdpi或drawable-hdpi中

#### FIT_END
放大或缩小图片到正好可以放入到空间中的大小，与**FIT_CENTER**的不同点在于，把图片居下（长大于宽是）或居右（框大于长时）显示。
![fitEnd-left](http://upload-images.jianshu.io/upload_images/1362430-a768799845e6e59a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) and  ![fitEnd-right](http://upload-images.jianshu.io/upload_images/1362430-02abfc92df75532a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
android:scaleType="fitEnd"
test_400x300.png在drawable-mdpi或drawable-hdpi中

#### FIT_START
与**FIT_END**基本相同，只是图片会居左或上显示。

#### FIT_XY
**不固定长宽比例**的缩小或放大图片，直到用图片把控件区域全部填满。
![fitXY](http://upload-images.jianshu.io/upload_images/1362430-8465944c365bd64f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
android:scaleType="fitXY"
test_400x300.png在drawable-mdpi或drawable-hdpi中

#### MATRIX
用矩阵的方式绘制，从控件的左上角开始，不缩放图片，与**CENTER**相似，不同点在于把图片的左上角对上控件的左上角显示，超出控件的部分不显示。

![matrix](http://upload-images.jianshu.io/upload_images/1362430-96c125ffad2327fe?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
android:scaleType="matrix"
test_400x300.png在drawable-hdpi中

#### 总结
以上，除**fitXY**长宽比例不固定外，其他5中scale方法长宽比例都固定。

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得易懂且有趣
个人博客：http://pengtao.me, GitHub地址：https://github.com/CPPAlien
