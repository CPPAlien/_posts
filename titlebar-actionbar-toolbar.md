title: 从Titlebar到Actionbar再到Toolbar
date: 2017-06-04
tags: [Android, 发展史]
categories: Android
---
#### 写在前面

bar在英语中有“门闩（shuān）”之意，我想大家一看到“闩”这个字就能马上联想到bar是啥了。当然Android中对bar的定义和门闩还是有不少区别的。Android中的bar有引导使用之作用，快速让用户了解当前页面的功能和相关操作等。本着这个原则就有了下面的bar的发展史。



#### 从Titlebar到Actionbar

在3.0之前，Android在UI设计上还是比较简单粗暴的，当时对这个bar的作用定义也极为简单，就是为显示当前页面的标题，名字都命名成了Titlebar，意为标题bar。对这个bar能做的操作也很单一，设置标题，再或者加个logo啥的，再有复杂的就得自己自定义View了。这个bar长相就如下图所示，可以说丑到爆了。和当时iOS漂亮的界面简直一个天一个地，那个时代可以说是Android的试水期，只追求功能可用，而其他方面就不管了。

![Titlebar](http://upload-images.jianshu.io/upload_images/1362430-23bd1008230eacb9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


当时Android在操作设计上是有物理返回按钮的，所以这个Titlebar连返回都不支持。而在那个设计往往都要对标iOS的时代，Android程序员只能自己去扩展这个功能单一的Titlebar了。自己辛苦得模仿实现iOS应用上那些漂亮的导航栏功能。

```
getWindow().setFeatureInt(Window.FEATURE_CUSTOM_TITLE, R.layout.custom_title_bar);
```

经过几年的发展，到Android 2.3版本，Android的各大功能基本稳定了，Android界开始对美和使用方便性有了强烈的呼声。于是2011年Google发布了Android 3.0版本，这个版本的Android在UI设计上进行了大改，除了引入了大家熟悉的Fragment外，也开始把Titlebar替换为Actionbar。Actionbar顾名思义，这个bar就不当当只是展示标题了，而是加入了Action（动作）。Android3.0的发布相当于Android一个试错的旧时代的结束，一个高速发展的新时代的开始，虽然Android3.0本身应用的范围很小。



#### Actionbar时代

Actionbar主要功能除了显示Title外，还提供了一致的导航和视觉体验，突出了应用的关键操作。比如你可以使用`SetDisplayHomeAsUpEnabled(true)`添加向上导航，这样在点击Actionbar左边的应用图标时会返回父视图。


![](http://upload-images.jianshu.io/upload_images/1362430-134e9cdca73eb062.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


你还可以使用Menu的方式来添加当前页面的操作按钮。


![](http://upload-images.jianshu.io/upload_images/1362430-177ed4fa1114eac1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/480)


 你甚至还可以在Actionbar中自定义自己的View，比如你可以


![](http://upload-images.jianshu.io/upload_images/1362430-7a84a1cc0996a3a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


或者在Actionbar中加入页面切换的功能，配合Fragment的机制


![](http://upload-images.jianshu.io/upload_images/1362430-ae20eb549d69f191.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


另外Actionbar还提供了样式设定，多屏幕适配等等功能。算是解决了Titlebar的痛点，Android在顶部导航这一块终于有了自己的风格，而不再唯iOS是瞻。



#### 啊哈，Toolbar

又随着时代的发展，到了2014年，Android5.0发布，这个版本的发布标志着Android又进入了一个新时代，快速爆发的时代过去了，从现在起，要变得优雅，要有自己统一的风格。所以在这个版本上Google推出了[Material Design](https://material.io/guidelines/)设计语言。然后Actionbar就被重整了，Actionbar虽方便使用，功能也很强大。但它最大的缺点就是不够灵活，比如它默认只能在顶部，要想把Actionbar移到屏幕其它位置就会非常麻烦，因为它在设计上是绑定在DecorView的。你可以到`com.android.internal.policy.DecorView`中看到相关源码，比如有下面这段，把Actionbar固定在了顶部，并且横向填充。

```
mShowPrimaryActionModePopup = new Runnable() {
	public void run() {
		mPrimaryActionModePopup.showAtLocation(
		mPrimaryActionModeView.getApplicationWindowToken(),
			Gravity.TOP | Gravity.FILL_HORIZONTAL, 0, 0);
		endOnGoingFadeAnimation();
		...
	}
};
```

为了满足Material Design优雅而灵活的设计原则，Google推出了更加灵活的Toolbar。你可以在一个Activity中声明多个Toolbar并且放在任意位置。并且你可以在自己的Toolbar中塞入任何你想要的View。Toolbar不是Actionbar的替代，Toolbar是Actionbar的布局方式的扩展，就像一个家庭的小孩长大了，它可以去外面干更多的事情，而也可以回家继续做一个晚辈。所以我们可以自己定义一个Toolbar，然后调用`setActionBar()`来让这个Toolbar充当Actionbar的功能，前提你需要把当前Activity主题或feature设为NoActionBar。

#### Toolbar的使用
这里只简单介绍下Toolbar的一种用法。关于Toolbar的详细用法，可以参考：[Google的官方文档](https://developer.android.com/training/appbar/index.html)。我们用向Toolbar中加入一个搜索按钮来举个例子。

首先你需要在布局中加入toolbar，可以如普通View一样放在布局的任意位置，并且在Activity的onCreate方法中设置该toolbar为actionbar。
```
<android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:background="@color/colorPrimary"
        android:layout_height="48dp"/>

setSupportActionBar((Toolbar) findViewById(R.id.toolbar));
```
然后创建一个包含搜索按钮的menu，内容如下：
```
<?xml version="1.0" encoding="utf-8"?>
<menu
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/action_search"
        android:icon="@android:drawable/ic_menu_search"
        android:title="Search"
        app:showAsAction="ifRoom"/>
</menu>
```
最后在对应的Activity中加入该menu，就能得到如下图所示的效果了。
```
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.test_menu, menu);
    return true;
}
```

![加入Action后的效果](http://upload-images.jianshu.io/upload_images/1362430-6731c762fc7e7ced.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*TIPS：有人可能注意到在使用showAsAction时使用了app作为命名空间，而非使用android。这个是一个历史遗留问题，因为ActionBar是在Android 3.0上引入的，它也具有android:showAsAction这个属性，所以appcompat-v7包为了向下兼容，就用了自定义属性来避免冲突。*



原文地址：http://pengtao.me/2017-06-04/titlebar-actionbar-toolbar/

>**作者简介**
>CPPAlien([@彭涛me](http://weibo.com/creaspan))   如有疑问，你可以通过以下方式联系我
>Blog：http://pengtao.me
>Github：https://github.com/CPPAlien
>简书：http://www.jianshu.com/u/f9246f41945e