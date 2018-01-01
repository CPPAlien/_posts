title: Activity重建时保持Fragment状态的方法
date: 2016-02-18
tags: [Android, Fragment]
categories: Android
---
#### 假设场景
用Fragment方法实现一个居中的EditText，在EditText中输入一些内容，要求在屏幕旋转后，EditText中已经输入的内容不会被清空。

#### 1，不要重复创建Fragment
每次activity被销毁时，当前的fragment状态都会被自动保存，所以如果不用以下方法加以判断，那么每次activity重建都会重复产生fragment。
```
FragmentManager fm= getSupportFragmentManager();
Fragment fragment = fm.findFragmentById(R.id.second_fragment_container);
if ( fragment == null ) {
    fragment = new SecondFragment();
    fm.beginTransaction().add(R.id.second_fragment_container, fragment).commit();
}
```

#### 2，使用setRetainInstance方法
设置该方法为true后，可以让fragment在activity被重建时保持实例不变。
```
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setRetainInstance(true);
}
```

此方法设置后会让activity在重建时的fragment生命周期与activity生命周期产生一些差别。差别如下：

 1. onDestroy将不会被调用（但onDetach方法会，因为fragment将会先从当前activity中分离）
 2. onCreate将因为fragment没有被重新创建而不会被调用
 3. onAttach和onActivityCreated还将会被调用

#### 3，保存View为全局变量
有人会发现，为什么设置了setRetainInstance方法，但是旋转屏幕时，EditText中输入的内容也会被清除呢？这是因为Fragment的onCreateView方法被重新执行了，重新创建了一个新的View，自然以前输入的内容就没有了。此时你可以设置一个全局的View，方法如下：
```
View view = null;
@Nullable
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    if ( view == null ) {
        Log.e("TestFragment", "view == null");
        view = inflater.inflate(R.layout.fragment_second, container, false);
    }
    return view;
}
```
