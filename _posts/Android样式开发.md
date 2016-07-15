title: Android样式开发
date: 2016-07-15 09:58:16
tags:
---

#Shape
##矩形
```bash
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <!-- solid指定形状的填充色，只有android:color一个属性 -->
    <solid android:color="#2F90BD" />
</shape>
```
```bash
//添加内边距
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <!-- solid指定形状的填充色，只有android:color一个属性 -->
    <solid android:color="#2F90BD" />
    <!-- padding设置内容区域离边界的间距 -->
    <padding
        android:bottom="12dp"
        android:left="12dp"
        android:right="12dp"
        android:top="12dp" />
</shape>
//渐变
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <!-- solid指定形状的填充色，只有android:color一个属性 -->
    <solid android:color="#2F90BD" />
    <!-- padding设置内容区域离边界的间距 -->
    <padding
        android:bottom="12dp"
        android:left="12dp"
        android:right="12dp"
        android:top="12dp" />
    <!-- gradient设置渐变 -->
    <gradient
        android:angle="270"
        android:endColor="#CC2F90BD"
        android:startColor="#222F90BD" />
</shape
```

参照：https://github.com/guosen/kstyle

