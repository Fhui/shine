---
title: MaterialDesign设计风格之自定义toolbar的简单实现 
---

<img src="http://i.imgur.com/1pxVVXE.png" width = "600" height = "300" align=center />

<!--more-->

MaterialDesign是Google在2014 I/O年发布的一种设计风格. android5.0也是开始使用这种设计风格
英文文档：http://www.google.com/design/spec/material-design/
中文网站：http://wiki.jikexueyuan.com/project/material-design/
今天我们主要学习toolbar，还有Google提供的侧滑控件，DrawerLayout，顺带把Google提供的下拉刷新控件SwipeRefreshLayout举例说明下。
![这里写图片描述](http://img.blog.csdn.net/20160721095729980)


效果如下
![这里写图片描述](http://img.blog.csdn.net/20160721102037992)
上图就是借用的代码家大神的[api接口](http://gank.io/api)写了一个小demo。用的volley请求，添加缓存，imageloader图片加载，recycleview瀑布流，单一activity，fragment＋viewpager设计模式。有点跑题，有想要源码的可以取下面查看下载连接，交流学习开源为主。

因为toolbar是在v7包中，我们要添加依赖

```
compile 'com.android.support:appcompat-v7:24.0.0'
```

首先在自定义toolbar之前一定要把android自带的标题栏给隐藏掉，在theme里面添加行代码
```
  <item name="windowNoTitle">true</item>
```
然后单独写一个toolbar的布局，因为android自带的标题栏实在是丑爆了，怎么都不好搭配。于是乎，Google推出了设计风格，自定义toolbar，简直是凤姐变志玲啊。
Google把toolbar放在了supportv7包里面，见代码。

```
<android.support.v7.widget.Toolbar
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:id="@+id/custom_toolbar"
    android:background="@android:color/holo_green_dark"
    android:popupTheme="@style/ThemeOverlay.AppCompat.Light"
    app:theme="@style/ThemeOverlay.AppCompat.ActionBar">

</android.support.v7.widget.Toolbar>
```
然而一定要在style里面设置主题添加上toolbar的主题

```
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
  <!-- Customize your theme here. -->
  <item name="colorPrimary">@color/colorOrange</item>
  <item name="colorPrimaryDark">@color/colorOrange</item>
  <item name="colorAccent">@color/colorOrange</item>
  <item name="windowNoTitle">true</item>
  <item name="android:textColorPrimary">@android:color/white</item>
<item name="drawerArrowStyle">@style/AppTheme.DrawerArrowToggle</item>
</style>
```

```

    <style name="AppTheme.DrawerArrowToggle"
           parent="Base.Widget.AppCompat.DrawerArrowToggle">
        <item name="color">@android:color/white</item>
    </style>
```
然后我们开始写DrawerLayout这个就比较简单了，我们可以作为根布局

```
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/dl_left"
    android:layout_width="match_parent"
    android:layout_height="match_parent">



   <LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <android.support.v4.view.ViewPager
            android:id="@+id/vp_home"
            android:layout_width="match_parent"
            android:layout_height="match_parent">

        </android.support.v4.view.ViewPager>

    </FrameLayout>
</LinearLayout>


    <FrameLayout
        android:layout_width="280dp"
        android:layout_height="match_parent"
        android:id="@+id/fl_left"
        android:layout_gravity="left"
        android:background="@android:color/white">

    </FrameLayout>

</android.support.v4.widget.DrawerLayout>

```

这里我们要注意下，drawerlayout中侧滑的布局要放在后面，如果写到了侧滑的布局，要设置这个属性`android:layout_gravity="left"`设置是哪边侧滑，也是一个标记。
这样基本就搞定了，然后设置一些属性直接找到该空间，设置一些文字之类的。非常简单。
当然也可以在style里面设置。是不是感觉非常的炫酷？
下面说一下`SwipeRefreshLayout`官方提供的下拉刷新控件。
这个控件是在v4兼容包里面。
也可以作为根布局来使用，上代码。

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.SwipeRefreshLayout
              xmlns:android="http://schemas.android.com/apk/res/android"
              android:id="@+id/swip_layout"
              android:layout_width="match_parent"
              android:layout_height="match_parent">


        <ListView
            android:id="@+id/ganhuo_lv"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:divider="@null"
            android:dividerHeight="10dp"
            android:padding="5dp">

        </ListView>


</android.support.v4.widget.SwipeRefreshLayout>
```

在要下拉刷新的activity中实现这个监听接口

```
 SwipeRefreshLayout.OnRefreshListener
```
重写`onRefresh`方法，我是用的handler来更新的ui布局

```
    @Override
    public void onRefresh() {
        Message msg = Message.obtain();
        msg.what = 1;
        handler.sendMessage(msg);
    }

```
里面可以做一些获取数据的操作啊，我这儿简单起见，并没有模拟。

```
  private Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            switch(msg.what){
                case 1:
                    swip_layout.setRefreshing(false);
                    break;
            }
            super.handleMessage(msg);
        }
    };
```
用起来也是特别的简单，而且这个下拉刷新控件还支持设置不同的颜色

```
swip_layout.setColorSchemeResources(android.R.color.holo_blue_light, android.R.color.holo_orange_dark,
                android.R.color.holo_green_light, android.R.color.holo_red_dark);
```

最后，有想看源码的可以戳此连接，https://github.com/Fhui/GankApp
个人网站：http://nexthiman.com





