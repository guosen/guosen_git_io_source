title: Android Activity的详解
date: 2016-03-15 11:30:34
tags:
- Android
- Activity
---
# Activity的生命周期
## 在Android中Activity状态分为：1.激活态（Active）一个新的Activity入栈后，展示屏幕的最前端，可以与用户交互；2.OnPause：当一个Activity被一个透明或者Dialog样式的Activity给覆盖时的状态，依然可见，但是失去焦点，不能与用户交互；
3.onStop:失去焦点（同样是被覆盖、不可见）
4.Killed：被系统杀死回收或者没有被启动。
```bash
   @Override
    protected void onStart() {
        super.onStart();
        // onCreate() 方法之后被调用，或者在 Activity 从 Stop 状态转换为 Active 状态时被调用.
    }
    @Override
    protected void onStop() {
        Log.i(Tag,"OnStop=====================");//Activity变成不可见 失去焦点
        super.onStop();
        //Activity 从 Active 状态转换到 Stop 态时被调用。一般我们在这里保存 Activity 的状态信息
    }
    @Override
    protected void onPause() {
        Log.i(Tag,"onPause=====================");//Activity状态依然可见 失去jiaodian不能交互//1
        super.onPause();
    }
    @Override
    protected void onResume() {
        Log.i(Tag,"onResume=====================");
        super.onResume();
        //Activity 从 Pause 状态转<--> Active 状态时被调用。
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
    }
```
# Activity栈
## Android 是通过一种 Activity 栈的方式来管理 Activity 的，一个 Activity 的实例的状态决定它在栈中的位置。处于前台的 Activity 总是在栈的顶端，当前台的 Activity 因为异常或其它原因被销毁时，处于栈第二层的 Activity 将被激活，上浮到栈顶。当新的 Activity 启动入栈时，原 Activity 会被压入到栈的第二层。一个 Activity 在栈中的位置变化反映了它在不同状态间的转换。

# Activity 的 Intent Filter
## 
```bash
 <intent-filter > 
 <action android:name="android.intent.action.MAIN" /> 
 <action android:name="com.zy.myaction" /> 
 </intent-filter>
```
