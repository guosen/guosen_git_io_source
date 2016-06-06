title: Android 消息机制Handle-Looper-Message
date: 2016-03-04 17:03:48
tags:
- Android
- Handle
- 消息机制
---
# Handler
## Handler这个类可以说是消息的处理者他有几个个构造函数：
1. publicHandler() 
2. publicHandler(Callbackcallback) 
3. publicHandler(Looperlooper) 
4. publicHandler(Looperlooper, Callbackcallback) 

```bash
    //实现这个函数来接收消息
    public void handleMessage(Message msg) {
    }
```
能够处理Message，我们有两种办法： 
1. 向Hanlder的构造函数传入一个Handler.Callback对象，并实现Handler.Callback的handleMessage方法 
2. 无需向Hanlder的构造函数传入Handler.Callback对象，但是需要重写Handler本身的handleMessage方法
所以一般的用法就是：
``bash
Handler handler=new Handler(Looper.myLooper){
	handleMassage(Message obj){

}
};
``
# Looper 循环者
## Looper字面上意思就是循环者，我们知道MessageQueue就是一个消息队列，就是存储Message的地方，一个队列摆在那只是静止的，那么怎么让他动起来发挥它原本应该有的作用呢。这时Looper就出来了，它可以说睡是一个动力让队列动起来！
```bash
class LooperThread extends Thread {
      public Handler mHandler;
      public void run() {
          Looper.prepare();

          mHandler = new Handler() {
              public void handleMessage(Message msg) {
                  // process incoming messages here
              }
          };
          Looper.loop();
      }
  }
```
将当前线程初始化为一个Looper，源码如下：
```bash
public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```
我们在用的时候应该是：在run方法里面，调用prepare（）和loop（）方法；
一个线程只能有一个Looper。

# Message
## 顾名思义 消息，就是传递的对象