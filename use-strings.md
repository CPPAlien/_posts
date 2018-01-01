title: 你真的会用Android中Strings资源吗
date: 2016-02-29
tags: [Android, 资源]
categories: Android
---
Android为了帮助开发者把应用更方便发布给全球不同语言的人们使用，建议开发者在进行开发时不要把UI呈现相关的文本内容硬编码，而是把内容写入到strings.xml中，这样做更加灵活，也更方便翻译成不同其他语言。下面通过一个案例来逐步介绍一下strings的用法。

#### 基本用法
比如我想在我的应用页面上呈现一句话，叫“我想买一台Kindle”，那就可以在strings.xml中加入如下：
```
<string name="buy_kindle">我想买一台Kindle</string>
```
在需要使用该字符串的地方通过getString获得后使用。
```
getString(R.string.buy_kindle)
```

#### 添加参数
现在你觉得这句话把买Kindle的数量写死了，因为你在代码中不同地方都用到了这句话，但所买的Kindle数量不一样。strings资源让开发者可以自主在字符串的任何位置加上参数，比如要解决这个问题，可以：
```
<string name="one">一</string>
<string name="buy_kindle">我想买%s台Kindle</string>
```
在有参数的情况下可以通过如下方式传入
```
getString(R.string.buy_kindle, getString(R.string.one))
```
如果你想在这句话中加入多个参数，比如想说：“我想买一台Kindle送给小明”，而送给谁可以自定，则可以如下表达。
```
<string name="buy_kindle">我想买%1$s台Kindle送给%2$s</string>
getString(R.string.buy_kindle, getString(R.string.one), getString(R.string.xiaoming))
```
注意在多个参数时，需要给参数加入位置信息，如上的%1$s。后面加上的参数会根据位置信息对应入号。具体做法如Java中的formatter方法一样，请参考：
http://developer.android.com/reference/java/util/Formatter.html

#### 国际化
比如想把这句话翻译成英文，你可以新创建一个英文的Values resource file，如下图方式选择：
![Android Studio中创建图示](http://upload-images.jianshu.io/upload_images/1362430-53194070e13fe9b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
翻译成如下：
```
<string name="buy_kindle">I want to buy %1$s Kindle</string>
```

从中可以发现Kindle这个词并没有翻译，Android中对无需翻译的词，我们可以用<xliff:g>标签来标注起来，这样我们把资源文件给他人或者使用Google Play自动翻译服务时，对方就知道该部分无需翻译。如下：
```
<resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
<string name="buy_kindle">我想买%s台<xliff:g id="Kindle">Kindle</xliff:g></string>
```
注意使用该标签时，先在资源之前加上命名空间：
xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2"。

*TIPS：Google play控制台提供APP TRANSLATION SERVICE的翻译服务*

#### 带数量判定的字符串
在翻译成英语后，发现问题来了，如果我想表示买了多台Kindle，但在英语语法中多台Kindle需要用复数形式Kindles，这要如何做呢？Android为这种情形提供了Plurals方法。
```
<plurals name="buy_kindle">   
    <item quantity="one">I want to buy a Kindle</item>    
    <item quantity="other">I want to buy some Kindles</item>
</plurals>
```
获得该plurals方法如下，第二参数传入quantity，系统会根据quantity来选择对应的显示，该方法后也可以加入参数：
```
getResources().getQuantityString(R.plurals.buy_kindle, 2)
```
中文部分可以改成：
```
<plurals name="buy_kindle">    
    <item quantity="one">我想买一台<xliff:g id="Kindle">Kindle</xliff:g></item>    
    <item quantity="other">我想买几台<xliff:g id="Kindle">Kindle</xliff:g></item>
</plurals>
```

*TIPS：关于Quantity String的更详细说明请移步：
http://developer.android.com/guide/topics/resources/string-resource.html#Plurals*

#### 加入特殊字符
有些字符是没有办法在strings.xml里面直接写的，比如"<"，">"，但可以用它对应的ASCII码来替代进行显示，比如要表达：我想买一台Kindle<$100>，则可以：
```
<item quantity="one">我想买一台<xliff:g id="Kindle">Kindle<$100></xliff:g></item> 
```
其中“<”的ASCII是&#060，“>”的是&#062。更多特殊字符与ASCII对应表可以查看：[《常见字符与ASCII十进制对应表》](http://www.jianshu.com/p/150f6c9e450a)

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得简单而有趣
个人博客：http://pengtao.me, GitHub地址：https://github.com/CPPAlien
