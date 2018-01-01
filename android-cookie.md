title: Android端实现Cookie机制
date: 2016-02-19
tags: [Android, Http]
categories: Android
---
### 简介
Session是服务端验证客户端身份的一种机制。而Cookie是客户端存储的一种身份凭证，由服务端在回应的消息头中通过Set-Cookie字段“种”在客户端。以后每次客户端在向服务端请求时都会在消息头中带上Cookie字段。服务端就会根据Cookie的来判断此次请求是从哪个用户发过来的，是否是一次有效请求等。有关Cookie的标准定义可以参考 [RFC6265](https://tools.ietf.org/html/rfc6265)。以下我们稍做介绍。

### 举个例子
首次打开浏览器请求http://www.baidu.com ，我们会得到一个response消息头如下格式。
![首次请求百度返回的消息头的一部分](http://upload-images.jianshu.io/upload_images/1362430-9c8ea207aabd8b0b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到里面包含了多条Set-Cookie，Cookie是key-value形式的。每条Cookie都有一个效信息字段，有些还含有expires、domain和path等字段。其中的domain和path字段，区分了不同的cookie可以被哪些网页链接获得，如未设置domain，则使用当前的访问链接替代。比如上面的BD_HOME=0，我们在下表[本地cookie信息]中，一样可以看到它的domain为www.baidu.com，这是因为我们访问的就是这个地址。其中的expires和max-age定义了该Cookie的有效期，失效的Cookie在下次请求时不会被加入到Cookie头中。

请求后我们查看浏览器的Cookie表，可以看到这些Cookie信息被“种”下了。
![本地cookie信息](http://upload-images.jianshu.io/upload_images/1362430-3a1a34406b9d7949?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时再发送一条请求，可以看到请求的消息头如下：
![请求消息头Cookie信息](http://upload-images.jianshu.io/upload_images/1362430-b65d2d3117dd987c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不同网站的请求，会因为请求链接的不同，只能从浏览器取得属于自己的cookie，根据上文提到的domain和path字段来区分。关于匹配的详细规则还是可以查看[RFC6265](https://tools.ietf.org/html/rfc6265)。这边我们用domain的匹配来稍微说下，在标准的5.1.3条目是有关`domain matching`的。
```
A string domain-matches a given domain string if at least one of the
   following conditions hold:

   o  The domain string and the string are identical.  (Note that both
      the domain string and the string will have been canonicalized to
      lower case at this point.)

   o  All of the following conditions hold:

      *  The domain string is a suffix of the string.

      *  The last character of the string that is not included in the
         domain string is a %x2E (".") character.

      *  The string is a host name (i.e., not an IP address).
```
从中我们可以看出domain要满足两点才能算是匹配，第一点，完全相同；如果第一点不满足则需要，请求链接的后缀包含存储的cookie的domain，并且不包含部分的最后一个字符还得是"."，并且还是是个域名，而非IP地址。domain匹配成功后，还要匹配path，path的匹配要比domain复杂一些，具体可以查看标准的5.1.4。path匹配可以做到一个网站下的页面可以分别存储不同的cookie，也可以共享上层父页面的cookie，比较灵活。

### 客户端实现
Android自身所带的HttpUrlConnection方法是默认不开启Cookie存储的。不过我们可以用java提供的几个类来在Android中实现：
可以先在所有请求之前声明
```
CookieHandler.setDefault(new CookieManager(null, CookiePolicy.ACCEPT_ALL));
```
开启此开关后，每次请求的Set-Cookie信息都会被CookieManager处理。CookieManager又会使用第一个参数传入的CookieStore来处理Cookie的存储问题，因为此处传入了null，系统会默认调用一个基于CookieStore实现的CookieStoreImpl类来处理Cookie的存储，这个类的只有基于内存的存储，当进程被杀死后，下次再进入应用，保存的Cookie信息就会丢失。所以我们需要基于CookieStore这个接口实现一个具有内存和本地双存储机制的Cookie存储类。

可以参考：Fran Montiel实现的PersistentCookieStore 类：
https://gist.github.com/franmontiel/ed12a2295566b7076161

当解决了Cookie的存储后，我们就需要考虑以后我们的每次请求需要在请求的消息头中加入Cookie字段。以上用CookieStore存储下来的Cookie信息都会被保存成HttpCookie形式的信息。我们可以上面的百度例子中看到Cookie的组成样式，所以我们可以提取CookieStore中的信息并组合。
```
StringBuilder cookieBuilder = new StringBuilder();
String divider = "";
for (HttpCookie cookie : getCookies()) {
    cookieBuilder.append(divider);
    divider = ";";
    cookieBuilder.append(cookie.getName());
    cookieBuilder.append("=");
    cookieBuilder.append(cookie.getValue());
}
cookieString = cookieBuilder.toString();
```
然后把这个cookieString在以后的请求中加入到头中，如果你用HttpUrlConnection，你就可以
```
setRequestProperty("Cookie",cookieString);  
```
如果你需要让应用中打开的WebView页面也能共享使用Cookie，则需要使用`android.webkit.CookieManager`类来设置，简单式例代码如下。注意，第一个参数要使用链接的host部分。这样让web端的不同页面也可以共享这些cookie。
```
for (HttpCookie cookie : getCookies()) {
    CookieManager.getInstance().setCookie(Uri.parse(url).getHost(), cookie.toString());
}
```
OK，大功告成。

以下是我在Github上开源的一个基于Volley实现的网络层框架，也包括Cookie机制的Http请求，欢迎大家fork：
https://github.com/CPPAlien/DaVinci

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得易懂且有趣
个人博客：http://pengtao.me, GitHub地址：https://github.com/CPPAlien
