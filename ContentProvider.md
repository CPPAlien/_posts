title: ContentProvider从入门到精通
date: 2016-05-02
tags: [Android, ContentProvider]
categories: Android
---
#### 前言
ContentProvider虽然与Activity、Service、BroadcastReceiver齐名为Android四大组件。但如果你不是特别开发一款与其他APP有数据交互的应用，它的使用频率远没有另外三者高。进而有些开发者可能在做过几个成熟应用后，对ContentProvider的理解还是不够深入，无法独立完成ContentProvider功能的开发。网上博客对这一块的内容介绍的也是比较复杂，不适合初学者研究学习，此篇希望能全面介绍下ContentProvider，从ContentProvider在框架中所充当的角色，到ContentResolver的使用，到URI的概念，再到数据共享的方法和权限管理，一步步的让大家对ContentProvider有个全面的认识。

#### ContentProvider的角色
ContentProvider一般为存储和获取数据提供统一的接口，可以在不同的应用程序之间共享数据。

之所以使用ContentProvider，主要有以下几个理由：
1，ContentProvider提供了对底层数据存储方式的抽象。比如下图中，底层使用了SQLite数据库，在用了ContentProvider封装后，即使你把数据库换成MongoDB，也不会对上层数据使用层代码产生影响。

![ContentProvider角色](http://upload-images.jianshu.io/upload_images/1362430-1857f913ab2e7a3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


2，Android框架中的一些类需要ContentProvider类型数据。如果你想让你的数据可以使用在如SyncAdapter, Loader, CursorAdapter等类上，那么你就需要为你的数据做一层ContentProvider封装。

3，第三个原因也是最主要的原因，是ContentProvider为应用间的数据交互提供了一个安全的环境。它准许你把自己的应用数据根据需求开放给其他应用进行增、删、改、查，而不用担心直接开放数据库权限而带来的安全问题。

我们知道了ContentProvider是对数据层的封装后，那么大家可能会问我们要如何对ContentProvider进行增，删，改，查的操作呢？下面我们来介绍一个新的类ContentResolver，我们可以通过它，来对不同的ContentProvider进行操作。

#### ContentResolver
有些人可能会疑惑，为什么我们不直接访问Provider，而是又在上面加了一层ContentResolver来进行对其的操作，这样岂不是更复杂了吗？其实不然，大家要知道一台手机中可不是只有一个Provider内容，它可能安装了很多含有Provider的应用，比如联系人应用，日历应用，字典应用等等。有如此多的Provider，如果你开发一款应用要使用其中多个，如果让你去了解每个ContentProvider的不同实现，岂不是要头都大了。所以Android为我们提供了ContentResolver来统一管理与不同ContentProvider间的操作。


![ContentResolver角色](http://upload-images.jianshu.io/upload_images/1362430-a336044d52818448.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在[Context.java](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/java/android/content/Context.java)的源码中有一段
```
/** Return a ContentResolver instance for your application's package. */
 public abstract ContentResolver getContentResolver();
```
所以我们可以通过在所有继承Context的类中通过调用`getContentResolver()`来获得`ContentResolver`。 

可能又有童鞋会问，那ContentResolver是如何来区别不同的ContentProvider的呢？这就涉及到URI（Uniform Resource Identifier）问题，对URI是什么还不明白的童鞋请自行Google。

#### ContentProvider中的URI
ContentProvider中的URI有固定格式，如下图：

![URI](http://upload-images.jianshu.io/upload_images/1362430-b39bc91ec8e272af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Authority：**授权信息，用以区别不同的ContentProvider；
**Path：**表名，用以区分ContentProvider中不同的数据表；
**Id：**Id号，用以区别表中的不同数据；

URI组装代码示例：
```
public class TestContract {

    protected static final String CONTENT_AUTHORITY = "me.pengtao.contentprovidertest";
    protected static final Uri BASE_CONTENT_URI = Uri.parse("content://" + CONTENT_AUTHORITY);

    protected static final String PATH_TEST = "test";
    public static final class TestEntry implements BaseColumns {

        public static final Uri CONTENT_URI = BASE_CONTENT_URI.buildUpon().appendPath(PATH_TEST).build();
        protected static Uri buildUri(long id) {
            return ContentUris.withAppendedId(CONTENT_URI, id);
        }

        protected static final String TABLE_NAME = "test";

        public static final String COLUMN_NAME = "name";
    }
}
```
从上面代码我们可以看到，我们创建了一个
`content://me.pengtao.contentprovidertest/test`的uri，并且开了一个静态方法，用以在有新数据产生时根据id生成新的uri。下面介绍下如何把此uri映射到数据库表中。

#### 实作
首先我们创建一个自己的`TestProvider`继承`ContentProvider`。默认该Provider需要实现如下六个方法，`onCreate()`, `query(Uri, String[], String, String[], String)`,`insert(Uri, ContentValues)`, `update(Uri, ContentValues, String, String[])`, `delete(Uri, String, String[])`, `getType(Uri)`，方法的具体介绍可以参考
http://developer.android.com/reference/android/content/ContentProvider.html 

下面我们以实现insert和query方法为例
```
private final static int TEST = 100;

static UriMatcher buildUriMatcher() {
    final UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
    final String authority = TestContract.CONTENT_AUTHORITY;

    matcher.addURI(authority, TestContract.PATH_TEST, TEST);

    return matcher;
}

@Nullable
@Override
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
    final SQLiteDatabase db = mOpenHelper.getReadableDatabase();

    Cursor cursor = null;
    switch ( buildUriMatcher().match(uri)) {
        case TEST:
            cursor = db.query(TestContract.TestEntry.TABLE_NAME, projection, selection, selectionArgs, sortOrder, null, null);
            break;
    }

    return cursor;
}

@Nullable
@Override
public Uri insert(Uri uri, ContentValues values) {
    final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
    Uri returnUri;
    long _id;
    switch ( buildUriMatcher().match(uri)) {
        case TEST:
            _id = db.insert(TestContract.TestEntry.TABLE_NAME, null, values);
            if ( _id > 0 )
                returnUri = TestContract.TestEntry.buildUri(_id);
            else
                throw new android.database.SQLException("Failed to insert row into " + uri);
            break;
        default:
            throw new android.database.SQLException("Unknown uri: " + uri);
    }
    return returnUri;
}
```
此例中我们可以看到，我们根据path的不同，来区别对不同的数据库表进行操作，从而完成uri与具体数据库间的映射关系。

因为ContentProvider作为四大组件之一，所以还需要在AndroidManifest.xml中注册一下。
```
<provider    
    android:authorities="me.pengtao.contentprovidertest"  
    android:name=".provider.TestProvider" />
```

然后你就可以使用`getContentResolver()`方法来对该ContentProvider进行操作了，ContentResolver对应ContentProvider也有insert，query，delete等方法，详情请参考： 
http://developer.android.com/reference/android/content/ContentResolver.html 

此处因为我们只实现了ContentProvider的query和insert的方法，所以我们可以进行插入和查询处理。如下我们可以在某个Activity中进行如下操作，先插入一个数据`peng`，然后再从从表中读取第一行数据中的第二个字段的值。

```
ContentValues contentValues = new ContentValues();
contentValues.put(TestContract.TestEntry.COLUMN_NAME, "peng");
contentValues.put(TestContract.TestEntry._ID, System.currentTimeMillis());
getContentResolver().insert(TestContract.TestEntry.CONTENT_URI, contentValues);

Cursor cursor = getContentResolver().query(TestContract.TestEntry.CONTENT_URI, null, null, null, null);

try {
    Log.e("ContentProviderTest", "total data number = " + cursor.getCount());
    cursor.moveToFirst();
    Log.e("ContentProviderTest", "total data number = " + cursor.getString(1));
} finally {
    cursor.close();
}
```

#### 数据共享
以上例子中创建的ContentProvider只能在本应用内访问，那如何让其他应用也可以访问此应用中的数据呢，一种方法是向此应用设置一个`android:sharedUserId`，然后需要访问此数据的应用也设置同一个`sharedUserId`，具有同样的sharedUserId的应用间可以共享数据。

但此种方法不够安全，也无法做到对不同数据进行不同读写权限的管理，下面我们就来详细介绍下ContentProvider中的数据共享规则。

首先我们先介绍下，共享数据所涉及到的几个重要标签：
`android:exported` 设置此provider是否可以被其他应用使用。
`android:readPermission` 该provider的读权限的标识
`android:writePermission` 该provider的写权限标识
`android:permission` provider读写权限标识
`android:grantUriPermissions` 临时权限标识，true时，意味着该provider下所有数据均可被临时使用；false时，则反之，但可以通过设置`<grant-uri-permission>`标签来指定哪些路径可以被临时使用。这么说可能还是不容易理解，我们举个例子，比如你开发了一个邮箱应用，其中含有附件需要第三方应用打开，但第三方应用又没有向你申请该附件的读权限，但如果你设置了此标签，则可以在start第三方应用时，传入`FLAG_GRANT_READ_URI_PERMISSION`或`FLAG_GRANT_WRITE_URI_PERMISSION`来让第三方应用临时具有读写该数据的权限。

知道了这些标签用法后，让我们改写下AndroidManifest.xml，让ContentProvider可以被其他应用查询。

声明一个permission
```
<permission android:name="me.pengtao.READ" android:protectionLevel="normal"/>```
然后改变provider标签为
```
<provider
    android:authorities="me.pengtao.contentprovidertest"
    android:name=".provider.TestProvider"
    android:readPermission="me.pengtao.READ"
    android:exported="true">
</provider>
```

则在其他应用中可以使用以下权限来对TestProvider进行访问。
```
<uses-permission android:name="me.pengtao.READ"/>
```

有人可能又想问，如果我的provider里面包含了不同的数据表，我希望对不同的数据表有不同的权限操作，要如何做呢？Android为这种场景提供了provider的子标签`<path-permission>`，path-permission包括了以下几个标签。
```
<path-permission android:path="string"
                 android:pathPrefix="string"
                 android:pathPattern="string"
                 android:permission="string"
                 android:readPermission="string"
                 android:writePermission="string" />
```
可以对不同path设置不同的权限规则，具体如何设定我这里就不做详细介绍了，可以参考
http://developer.android.com/guide/topics/manifest/path-permission-element.html


#### 相关代码
ContentProviderTest
https://github.com/CPPAlien/ContentProviderTest

ContentResolverTest
https://github.com/CPPAlien/ContentResolverTest

*注：ContentResolverTest是读取ContentProviderTest中的数据来显示，所以需要先安装ContentProviderTest。*

>**作者简介**
彭涛([@彭涛me](http://weibo.com/creaspan)) 致力于让技术变得易懂且有趣
个人博客：http://pengtao.me, GitHub地址：https://github.com/CPPAlien
