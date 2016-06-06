title: Android 触摸事件分发详解（翻译）
date: 2016-04-13 15:58:42
tags:
---
#android 怎样处理触摸事件
## 1.每一个用户触摸事件都被当作是一个MotionEvent对象；
2.描述用户的当前行为
  -ACTION_DOWN
  -ACTION_UP
  -ACTION_MOVE
  -ACTION_POINTER_DOWN
  -ACTION_POINTER_UP
  -ACTION_CANCEL
3.数据包括
 -触摸的位置（x,y）
 -手指数
 -时间点
 4.手势（gesture）就是从ACTION_DOWN到ACTION_UP之间过程
 #android处理触摸事件流程
 1.事件的起始点是Activity的dispatchTouchEvent()
 2.事件是从上一层往下传递（ViewGroup->子View），并且可以在任意一层intercept（拦截事件）
 3.任何没有被消费的事件最终都会到Activity的onTouchEvent()去处理
 4.监听器onTouchListener可以拦截onTouch()z在任意一个View和viewGroup；
 #android处理触摸事各个流程
 1.Activity.dispatchTouchEvent()
 这个是最先被调用的
 分发事件到Root view（attch to window）
 2.view.onTouchEvent()
 -发送事件到监听（如果有view.onTouchListener.onTouch()）
 -如果没有消费会调用自己的View.onTouchEvent()
 3.viewGroup.dispatchTouchEvent()
 -oninterceptTouchEvent(检查是否粗腰传递给子View)
 -对于每个子控件，以相反的顺序分发事件
 -如果没有子控件处理事件，就会传到OnTouchListener.onTouch()
 -如果没有监听 onToucheEvent()
 4.Intercept (拦截事件)跳过子控件
 #自定义Touch事件
 1.重写onTouchEvent方法；
 2.消费事件（return true）
 3.有用的常量ViewConfiguration
 -getScaledTouchSlop()
 (拖动最短距离)
 -getScaledMinimumFlingVelocity(最少平滑距离)
 -getLongPressTimeout()
 判定为长按时间值；

 原文：http://trinea.github.io/download/pdf/android/PRE_andevcon_mastering-the-android-touch-system.pdf
 事例项目：https://github.com/guosen/custom-touch-examples



