title: Android 动画初探
date: 2016-04-20 10:56:53
tags:
---
#Tween动画
(Creates an animation by performing a series of transformations on a single image with an Animation)
- 缩放
- 平移
- 渐变
- 旋转
## xml实现
```bash
<?xml version="1.0" encoding="utf-8"?>
<set
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="together"
    android:duration="2000">
    <alpha
        android:fromAlpha="1.0"
        android:toAlpha="0.0"
        ></alpha>
    <scale
        android:fromXScale="0"
        android:fromYScale="0"
        android:pivotX="0.5"
        android:pivotY="0.5"
        android:toXScale="50dp"
        android:toYScale="50dp"/>
</set>
//alpha              渐变透明度动画效果
//scale              渐变尺寸伸缩动画效果
//translate         画面位置移动动画效果
//rotate              画面旋转动画效果
```
###调用动画
```bash
ImageView image = (ImageView) findViewById(R.id.image);
Animation hyperspaceJump = AnimationUtils.loadAnimation(this, R.anim.hyperspace_jump);
image.startAnimation(hyperspaceJump);
```

## 插值器
### 用于修改一个动画过程中的速率，可以定义各种各样的非线性变化函数，比如加速、减速等。
在Android中所有的插值器都是Interpolator 的子类，通过 android:interpolator 属性你可以引用不同的插值器

#Frame 动画
## 逐帧播放动画
```bash
<?xml version="1.0" encoding="utf-8"?>

<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
   android:oneshot=["true" | "false"] >
	<item
		android:drawable="@[package:]drawable/drawable_resource_name"
		android:duration="integer" />
</animation-list>

```
调用代码
```bash
ImageView rocketImage = (ImageView) findViewById(R.id.rocket_image);
rocketImage.setBackgroundResource(R.drawable.rocket_thrust);
rocketAnimation = (AnimationDrawable) rocketImage.getBackground();
rocketAnimation.start();</span>
```

#参考文献
http://www.lightskystreet.com/2015/05/23/anim_basic_knowledge/