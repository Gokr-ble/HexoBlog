---
title: Android底部导航栏实现
date: 2020-09-24 20:05:28
tags: skills
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2020-9-24.png
thumbnail: /gallery/thumbnails/2020-9-24.png
toc: true
comments: true
---

总结一下安卓底部导航栏的实现流程。

<!--more-->

导航栏效果图：

![](/gallery/pictures/2020-9-24/1.png)

## 方法一：多层嵌套LinearLayout

- 放置一个横向的LinearLayout；
- 在里面依次放置若干纵向的LinearLayout；
- 再在里面放置一个ImageView和一个TextView，分别显示图片和文字说明。

这种方式的问题是当布局嵌套过多时，可能会产生性能问题，引起卡顿。

## 方法二：RadioGroup+RadioButton

首先准备好适当尺寸的图片：

![](/gallery/pictures/2020-9-24/2.png)

然后编写布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!--最外层结构-->
    <FrameLayout
        android:id="@+id/fragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@+id/id_diverline" />

    <!--分割线-->
    <View
        android:id="@+id/id_diverline"
        android:layout_above="@+id/id_bottom_tags"
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:background="#C2C5CE"/>

    <!--内部结构-->
    <LinearLayout
        android:id="@+id/id_bottom_tags"
        android:layout_width="match_parent"
        android:layout_height="55dp"
        android:layout_alignParentBottom="true"
        android:background="@drawable/bt_tag_bg"
        android:orientation="horizontal">

        <!--互斥按钮组-->
        <RadioGroup
            android:id="@+id/radio_group"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center"
            android:background="@color/transparent"
            android:gravity="center"
            android:orientation="horizontal">

            <!--需要多少个标签就写多少个RadioButton-->
            <RadioButton
                android:id="@+id/id_nav_message"
                android:layout_width="0dp"
                android:checked="true"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:layout_weight="1"
                android:background="@color/transparent"
                android:button="@null"
                android:clickable="true"
                android:drawablePadding="2dp"
                android:drawableTop="@drawable/nav_menu_message"
                android:gravity="center"
                android:onClick="switchView"
                android:text="消息"
                android:textColor="@drawable/nav_text_color"
                android:textSize="10sp" />

            <RadioButton
                android:id="@+id/id_nav_personal"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:layout_weight="1"
                android:background="@color/transparent"
                android:button="@null"
                android:clickable="true"
                android:drawablePadding="2dp"
                android:drawableTop="@drawable/nav_menu_personal"
                android:gravity="center"
                android:onClick="switchView"
                android:text="我的"
                android:textColor="@drawable/nav_text_color"
                android:textSize="10sp" />

        </RadioGroup>
        
    </LinearLayout>

</RelativeLayout>
```

此处需要注意一下几点：

- `android:background`属性指定为自定义的透明背景；
- `android:checked`属性只在第一个标签上添加；
- `android:button="@null"`貌似是去掉RadioButton原生按钮的（不太清楚）；
- `android:onClick`属性指定在点击时响应哪个方法；
- `android:drawableTop`属性指定为自定义的图片；
- `android:textColor`指定为需要的字体颜色；

以下为个人使用的颜色，仅供参考：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#6200EE</color>
    <color name="colorPrimaryDark">#3700B3</color>
    <color name="colorAccent">#03DAC5</color>
    <color name="normal_btn_bg">#4674FD</color>
    <color name="selected_btn_bg">#0F46AF</color>
    <color name="transparent">#50FFFFFF</color>
    <color name="not_selected">#999999</color>
    <color name="selected">#D075EA</color>
</resources>
```

接下来实现`switchView`方法：

```kotlin
private fun changeFragment(fragment: Fragment) {
        val fragmentManager: FragmentManager = supportFragmentManager //开启事务
        val transaction: FragmentTransaction = fragmentManager.beginTransaction()
        transaction.replace(R.id.fragment, fragment)
        //transaction.addToBackStack(null)//是否使用返回栈
        transaction.commit()
    }

    fun switchView(view: View){
        when (view.id) {
            R.id.id_nav_message -> changeFragment(MessageFragment())
            R.id.id_nav_personal -> changeFragment(PersonalFragment())
        }
    }
```

然后自己再动态添加需要的Fragment就好了。