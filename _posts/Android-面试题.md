title: 2016 新浪微博 Android 面试题
date: 2016-04-14 16:11:50
tags:
---
# 静态内部类、内部类、匿名内部类，为什么内部类会持有外部类的引用？持有的引用是this？还是其它？
##
1.静态内部类：使用static修饰的内部类
2.匿名内部类：使用new生成的内部类
3.因为内部类的产生依赖于外部类，持有的引用是类名.this。
# ArrayList和Vector的主要区别是什么？
## ArrayList在Java1.2引入，用于替换Vector
# Vector:
##
 -线程同步
 -当Vector中的元素超过它的初始大小时，Vector会将它的容量翻倍
 #ArrayList:
 ##
 1.线程不同步，但性能很好
 2.当ArrayList中的元素超过它的初始大小时，ArrayList只增加50%的大小
 #Java中try catch finally的执行顺序
 ##先执行try中代码发生异常执行catch中代码，最后一定会执行finally中代码
 #switch是否能作用在byte上，是否能作用在long上，是否能作用在String上？
 ##switch支持使用byte类型，不支持long类型，String支持在java1.7引入
 #Activity和Fragment生命周期有哪些？
 ##
 ```bash
 Activity——onCreate->onStart->onResume->onPause->onStop->onDestroy
Fragment——onAttach->onCreate->onCreateView->onActivityCreated->onStart->onResume->onPause->onStop->onDestroyView->onDestroy->onDetach
 ```
#onInterceptTouchEvent()和onTouchEvent()的区别？
## onInterceptTouchEvent()用于拦截触摸事件
onTouchEvent()用于处理触摸事件
#RemoteView在哪些功能中使用
##APPwidget和Notification中
#SurfaceView和View的区别是什么？
##SurfaceView中采用了双缓存技术，在单独的线程中更新界面
View在UI线程中更新界面
#讲一下android中进程的优先级？
#一面
##service生命周期，可以执行耗时操作吗？
JNI开发流程
Java线程池，线程同步
自己设计一个图片加载框架
自定义View相关方法
http ResponseCode
插件化，动态加载
性能优化，MAT
AsyncTask原理
#二面
所使用的开源框架的实现原理，源码
没看过，被pass了
去面试之前把用到的开源框架源码分析一定要看看啊

