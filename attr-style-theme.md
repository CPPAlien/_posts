title: Attr、Style和Theme详解
date: 2016-05-11
tags: [Android, 资源]
categories: Android
---
![schnauzer.jpg](http://upload-images.jianshu.io/upload_images/1362430-8cf211b690ac6e53.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 前言
这三个概念贯穿Android框架的方方面面，是Android程序设计中很重要的一环，理解它们，并能学以致用，不但可以让你的代码变得简洁明了，还可以让你的应用更加灵活。但目前网上资料对这块介绍的知识点往往比较散，不是很系统全面，在此特以自己开发经验总结此文一篇，希望可以帮助初学者把这三个概念彻底搞明白，开发出高质量的Android代码。

#### 概念说明
**Attr：**属性，风格样式的最小单元；

**Style：**风格，它是一系列Attr的集合用以定义一个**View**的样式，比如height、width、padding等；

**Theme：**主题，它与Style作用一样，不同于Style作用于个一个单独View，而它是作用于Activity上或是整个应用。

#### Attr的定义
我们先举一个框架中的源码例子，用来介绍下Android中是如何定义一个Attr的，比如以下创建一个简单的TextView布局
![TextView](http://upload-images.jianshu.io/upload_images/1362430-c6e09469e9f1b771.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中`layout_width`对应到框架中的attr信息如下：
```
<declare-styleable name="ViewGroup_Layout">
	<attr name="layout_width" format="dimension">
	    <enum name="fill_parent" value="-1" />
	    <enum name="match_parent" value="-1" />
	    <enum name="wrap_content" value="-2" />
	</attr>
	...
</declare-styleable>
```
从上可以看到layout_width可以使用三个枚举值，并且其中fill_parent和match_parent的value值都为-1。做过Android开发的童鞋肯定知道，从2.2开始Android框架就推荐用match_parent代替fill_parent，而以上代码正实现了兼容，因为它们对应的值都为-1。

以上的textStyle的属性信息在源码中如下：

```
<attr name="textStyle">
    <flag name="normal" value="0" />
    <flag name="bold" value="1" />
    <flag name="italic" value="2" />
</attr>
```
它也对应了三个值，但这里却使用了flag标签。细心的童鞋可能已经明白了flag与enum的差别，flag表示这几个值可以做或运算，比如上面的textStyle，你可以叠加使用，如用`bold|italic`表示既加粗也变成斜体，而enum只能让你选择其中一个值。

看完上例后，我们来试着自己自定义一个自己的属性，在values目录下创建一个`attrs.xml`文件，在`<resources>`元素里面首先申明一个自己的`<declare-styleable>`表示一个属性组，再在里面加上属性就行。如下我们定义一个`DogStyle`的属性组，其中有三个属性一个是dogSex，一个是dogName，dogName的格式我们设置为`string`，最后一个是dogColor，这样一个属于我们自己的属性就定义成功了。


![DogStyle](http://upload-images.jianshu.io/upload_images/1362430-673719a0599319cd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

attr的format根据字面意思也挺容易理解的，这里我解释下`reference`的用法。它用在一些可以设置引用值的情况，比如`@drawable/myImage`、`@color/myColor`等。当然format也可以进行或运算，一般我们定义color类型的属性时，也一般会把format写成`format="reference|color"`，这样我们不但可以设置颜色值，如`#FFFFFF`，还可以使用我们自己定义的狗图片，如`@drawable/dog_pic`。

>TIPS：format即使用错，只要你自定义的View中获取对应类型值也是可以的，只是在布局中写代码时，IDE就不会根据你定义的format给出相应的提示了，所以最好在自定义View时还是仔细斟酌下类型。

#### Style的使用
如下我们在`styles.xml`中定义一个雪纳瑞风格
```
<style name="SchnauzerStyle">
    <item name="dogName">雪纳瑞</item>
    <item name="dogColor">@drawable/schnauzer</item>
    <item name="dogSex">boy</item>
</style>
```

下面我们看下如何让一个Style作用在一个View上的。
首先我们自定义了一个View命名为DogView，然后创建一个布局文件中加入该DogView视图，并让该View使用SchnauzerStyle风格。代码如下：
```
<cn.hadcn.test.DogView
    style="@style/SchnauzerStyle"
    android:layout_height="wrap_content"
    android:layout_width="wrap_content"/>
```
移步到DogView的Java代码中，我们可以通过theme的`obtainStyledAttributes`方法来获得我们刚刚定义的几个Attr属性在Style中的内容，如下我们举一个获得dogName的例子：
```
final Resources.Theme theme = context.getTheme();
TypedArray dogArray = theme.obtainStyledAttributes(attrs, R.styleable.DogStyle, defStyleAttr, defStyleRes);

String name = dogArray.getString(R.styleable.DogStyle_dogName);
Log.e("dog", "name = " + name);

dogArray.recycle();
```
以上`obtainStyledAttributes`有四个入参，前两个比较容易理解，后两个用作指定默认的Style，表示如果attrs中没有你想获得的属性，但如果你指定了默认Style，它会去从该默认的Style里面找你想要的属性。`defStyleAttr`和`defStyleRes`功能一样，指定的资源形式不同，前者表示一个默认的指向一个style风格的attr属性，而后者你可以直接传入一个style风格的id。注意以上定义的Style只能在这个DogView中被使用，如果你想在其他View使用，就需要再在需要使用的View中增加这个Style。这就是先前我们说的Style只能作用于一个View。


#### Theme的使用
Theme与Style使用同一个元素标签`<style>`，区别在于所包含的属性不同，并且使用的地方也不一样。Theme你需要设置到`AndroidManifest.xml`的`<application>`或者`<activity>`标签下，设置后，被设置的Activity或整个应用下所有的View都可以使用该`<style>`里面的属性了。

比如在上例中，我们直接把`SchnauzerStyle`设置到`<activity>`标签中，并把布局文件中DogView元素的`  style="@style/SchnauzerStyle"`栏位删除，以此来测试下，这个Activity下的所有View是不是可以直接使用theme中声明的这些属性。
```
<activity
    android:name=".MainActivity"
    android:theme="@style/SchnauzerStyle">
    ...
```

以上理论上是可行的，不过运行后，程序却出现奔溃，出现以下错误提示：
```
java.lang.IllegalStateException: You need to use a Theme.AppCompat theme (or descendant) with this activity.
```
有些同学一眼可能就看出，因为在这里Activity或Application的需要很多属性才能工作的，而此处我们只给它传一个`SchnauzerStyle`，这当然不行，所以我们需要对这个Style做下处理，让`SchnauzerStyle`继承一个系统主题，如下：
```
<style name="SchnauzerStyle" parent="Theme.AppCompat">
    <item name="dogName">雪纳瑞</item>
    <item name="dogColor">@drawable/schnauzer</item>
    <item name="dogSex">boy</item>
</style>
```
这样一个雪纳瑞主题就诞生了，而在这个Activity下的所有View都可以用雪纳瑞的信息了。Application中定义theme的原理一样，这里就不多说了。

>TIPS：框架使用Attr的顺序是：View中的Style会优先于Activity中的Theme，Activity中的Theme会优先于Application中的Theme，所以说你可以定义整个应用的总体风格，但局部风格你也可以做出自己的调整。

#### Attr的获得方法
有些情况下，我们可能需要使用theme中的属性值，比如下面我们想让一个TextView直接显示dogName这个属性的内容，并且使用系统字体的颜色，则可以如下做：
```
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textColor="?android:textColorSecondary"
    android:text="?attr/dogName"/>
```

获得一个Attr的方法，不同于普通资源使用`@`符号获得的方式，而是需要使用`?`符号来获得属性，整体的表达方式如下：
```
?[*<package_name>*:][*<resource_type>*/]*<resource_name>*
```
如果是本应用中的attr使用，则可以省去`<package_name>`部分。

此处的textColor使用当前主题的`android:textColorSecondary`属性内容。因为资源工具知道此处是一个属性，所以省去了attr （完整写法：`?android:attr/textColorSecondary`）。

#### 总结
我刚开始学Android的时候，也总对这三个概念很迷惑，不知道什么是属性，什么是风格，什么是主题，它们之间又有什么关系？它们在Android框架中又充当什么角色？又如何自己去定义？但随着学习的深入，越发觉得这三块内容真是Android框架的一大神器，有时你不用改动代码，只要换一个theme，应用马上焕发青春。而且也尝试用所学内容去写自己的theme，不但可以让自己的布局文件更加清晰明了，而且还让自己的代码具有更高的扩展性，真是好处多多，希望对这块还不了解的童鞋多多研习。

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得易懂且有趣
个人博客：http://pengtao.me, GitHub地址：https://github.com/CPPAlien
