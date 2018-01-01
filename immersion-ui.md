title: Android沉浸式UI，看这一篇就够了
date: 2017-08-06
tags: [Android, UI]
categories: Android
keywords: 安卓沉浸式
---
### 前言

Android对这种沉浸式风格的支持跨度了好几个版本，真正系统级别的完全支持要到Android 6.0，而从Android 3.0开始就多多少少有了点这种沉浸式风格的影子。从3.0到6.0，期间跨度了五个大版本，每个版本都多多少少做了些优化，加入了些新特性。所以如果我们想要一个比较完美的沉浸式实现，还得能尽可能支持低版本，将会是个很有挑战性的任务，期间会遇到各种各样兼容性问题。为了让大家少走弯路，下面我会按照版本的升级顺序，一点点说明每个版本中加入的新特性，让大家知道这种沉浸式风格是怎么一点点发展来的。

### 3.0（API 11）

大家还记得我们在Android 3.0以前代码上是如何隐藏状态栏的吗？我想早前做过Android开发的应该都对下面这段代码比较熟悉吧。

```
getWindow().addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
```

在3.0之前，我们并没有一个专门针对状态栏进行操作的API。而如果我们想要要隐藏状态栏来达到全屏效果，只能通过对Window设置全屏标签获得。后来在Android 3.0版本（API 11）上，View中加入了一个`void setSystemUiVisibility (int visibility)` 方法。随着该方法一同出来的有两个属性：`STATUS_BAR_HIDDEN`、`STATUS_BAR_VISIBLE`。并且还加入了`View.OnSystemUiVisibilityChangeListener`来监听系统UI的变化。至此Android开启了可以对状态栏进行操作的时代，也为以后的各种花样玩法打下了基础。不过在3.0上你能操作的也只有显示或隐藏状态栏了，并无其他。

### 4.0（API 14）

后来在4.0上，扩充了针对状态栏操作的功能，把3.0种加入的`STATUS_BAR_HIDDEN` 换成了`SYSTEM_UI_FLAG_LOW_PROFILE` ， `STATUS_BAR_VISIBLE` 也换成了 `SYSTEM_UI_FLAG_VISIBLE`。另外还加入了`SYSTEM_UI_FLAG_HIDE_NAVIGATION`这个新标签。开发者对系统UI的控制权进一步加强。

**SYSTEM_UI_FLAG_LOW_PROFILE** 被称为低调模式，我们可以设置该标签让状态栏或导航栏上的图标变暗和让一些不重要的图标消失。适用一些全屏操作，比如游戏，阅读等。我们来看下设置该标签后的效果。左边为默认样式，右边为设置该标签后的样子，可以明显看到状态栏一些ICON被隐藏了，并且变得有些半透明，导航栏的三个按钮也都被隐藏了。另外只要页面有任何交互，该标签会被清除，样式还原。
![](http://upload-images.jianshu.io/upload_images/1362430-6ae929ce43b4f962.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**SYSTEM_UI_FLAG_HIDE_NAVIGATION** 故名思意就是可以通过设置该标签，使得底部的导航栏隐藏。Android也正是从4.0版本开始可以对底部导航栏进行操作的。同样如果用户与页面有任何交互，该标签会被清除，样式会还原。

### 4.1（API 16）

在4.1版本中，Android团队进一步强化了`setSystemUiVisibility`的作用，又在4.0的基础上增加了以下一些属性：

**SYSTEM_UI_FLAG_FULLSCREEN** 全屏状态，视觉上的作用和`WindowManager.LayoutParams.FLAG_FULLSCREEN`一样。如果你的应用只是某几个页面需要使用全屏模式，建议使用该标签来设置。因为此标签更容易清除。

**SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN** / **SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION**

这两个属性和上面提到的FULLSCREEN和HIDE_NAVIGATION属性很像，只是多了一个LAYOUT。区别是，这两个属性并不会真正隐藏状态栏或者导航栏，只是把整个content的可布局区域延伸到了其中。如下所示Toolbar的布局已经延伸到了状态栏下面。


![](http://upload-images.jianshu.io/upload_images/1362430-94f5ebedd5031cd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**SYSTEM_UI_FLAG_LAYOUT_STABLE** 此View一般和上面几个提到的属性一起使用，它可以保证在系统控件隐藏显示时，不会让本view重新layout。也就是你手动隐藏状态栏或导航栏时，所有的view也都待在本来的位置上不会动。



### 4.4（API 19）

针对以上提到的版本中的状态栏，我们只能进行显示或隐藏的操作。并不能完全达到我们想要的沉浸式效果。我们想要状态栏与下方页面内容的风格更加统一，而且该显示的状态也不能丢，如下所示。


![沉浸式](http://upload-images.jianshu.io/upload_images/1362430-4555b73e1e5ad083.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




从Android 4.4 开始，我们才能真正做到以上这种效果，在API 19上加入了以下两个风格属性，使用这两种属性，可以设置状态栏或者导航栏的背景为透明。
```
android:windowTranslucentStatus
android:windowTranslucentNavigation
```

设置这两个属性后，需要注意的是，它在Android 4.4 上的效果和 Android 5.0 和之后版本上，表现出的效果不太一样。我们来实地演示下，在values-v19目录下的styles.xml中加入以下代码。

```
<style name="AppTheme"  parent="Theme.AppCompat.Light.NoActionBar">
	<item name="android:windowTranslucentStatus">true</item>
	<item name="android:windowTranslucentNavigation">true</item>
</style>
```

并且设置AppTheme为应用主题，然后分别运行在Android 4.4和5.0上。我们会看到如下效果


![](http://upload-images.jianshu.io/upload_images/1362430-8f8461b9ff5fdf98.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们会看到在4.4上的效果和5.0上的不一样，在4.0上为全透明，但上部有些许阴影，而在5.0上则为均匀的半透明。这可能是因为Android团队在4.4上实现此种风格时，并没有考虑到状态栏的背景可能是白色的情况，而在4.4上我们又不能更改状态栏内容的颜色。到了5.0的时候他们发现了这个问题的存在，但可能发现的比较晚，没有时间加上修改状态栏内容颜色的特性，所以干脆把这种默认风格变为半透明，这样就避开了背景为浅色调的问题。以上纯属个人猜测。

除了上面两个属性外，另外在API 19中还加入了两个标签

**SYSTEM_UI_FLAG_IMMERSIVE** 正如前面所说的 `SYSTEM_UI_FLAG_HIDE_NAVIGATION`和`SYSTEM_UI_FLAG_FULLSCREEN`在用户与屏幕有任何交互时，都会被清除。过后如需隐藏目的，又得重新设置。而此标签正是防止这种情况而加入，设置此标签后，只有从屏幕上方下滑，或者从屏幕下方上滑时才会执行清除，其他普通交互不会变化。

**SYSTEM_UI_FLAG_IMMERSIVE_STICKY** 该标签与`SYSTEM_UI_FLAG_IMMERSIVE`作用差不多，只是该标签会让`SYSTEM_UI_FLAG_HIDE_NAVIGATION`和`SYSTEM_UI_FLAG_FULLSCREEN` 标签被短暂清除，而不是永久，一会儿后又会自动恢复。



### 5.0（API 21）

从第一次可以操作状态栏的3.0版本（2011年发布）到可以设置状态栏的颜色的5.0版本（2014年发布），已经过去了漫长的3年时间，这三年间，开发者门根本没法实现一个体验比较完美的沉浸式应用。到5.0版本，加入了以下两个属性。

```
android:statusBarColor
android:navigationBarColor
```

通过以上两个风格属性的设定，我们可以把状态栏和导航栏改变成任何你想要的颜色了。所以我们要在5.0上实现上面4.4开篇提到的沉浸式风格，那首先设置`SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`标签。

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
	getWindow().getDecorView().setSystemUiVisibility(
		View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN );
}
```

然后在风格中加入一条，让状态栏背景透明

```
<item name="android:statusBarColor">@android:color/transparent</item>
```


![](http://upload-images.jianshu.io/upload_images/1362430-513a2767a9755a2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不过这样做后，我们会发现一个问题，Toolbar上部分与状态栏重合了。要解决这个问题，我们就需要用到`fitSystemWindows`（对应方法：setFitsSystemWindows）了。此属性设置在view上，让该view在计算大小时可以考虑留给系统UI的空间。如下我们在布局的Toolbar中加入`android:fitsSystemWindows="true"`

```
<android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:background="#00AEFF"
        android:fitsSystemWindows="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
```

这样该Toolbar的上部分就会留出状态栏部分（和padding差不多），从而实现了该种沉浸式效果。至此我们算是可以实现沉浸式UI了。但Android却还留了个大坑给我们，那就是我们没法设置状态栏上的内容颜色，这会导致，如果你的页面风格为浅色调，那状态栏上的内容你就基本看不见了，而这个坑一直到6.0上才得到了解决，兼容之难，可想而知。

### 6.0（API 23）

6.0上加入了`SYSTEM_UI_FLAG_LIGHT_STATUS_BAR`标签和`android:windowLightStatusBar`风格属性，终于你可以改变状态栏中的内容颜色了。设置后，状态栏的内容会变成暗色调，这样即使在浅色的背景上显示，也一样能看清了，如下


![](http://upload-images.jianshu.io/upload_images/1362430-b777f40df150bc2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 后记

Android开发一大之痛就是兼容性问题，而如果你想开发一款比较好的具有沉浸式UI风格的APP，那你肯定会为Android的兼容性问题抓狂。一种功能，特性实现却被分散到了各个不同版本，即使Android新版本加上了新特性你也没法使用，因为Android版本新版本覆盖速度是出了名的慢。感觉这也正证明了Google的Android团队在技术的产品设计上还是有些欠缺的。

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得易懂且有趣
个人博客：http://pengtao.me
简书：http://www.jianshu.com/u/f9246f41945e
GitHub：https://github.com/CPPAlien