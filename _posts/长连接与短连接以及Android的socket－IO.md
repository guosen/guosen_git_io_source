title: 长连接与短连接以及Android的socket－IO
date: 2016-06-13 16:06:05
tags:
- Android
- Socket
- 长连接
---
# TCP/IP是什么？
## TCP/IP是个协议组，可分为三个层次：网络层、传输层和应用层。
    在网络层有IP协议、ICMP协议、ARP协议、RARP协议和BOOTP协议。
    在传输层中有TCP协议与UDP协议。
    在应用层有FTP、HTTP、TELNET、SMTP、DNS等协议。
# Socket是什么呢？
## Socket是应用层与TCP/IP协议族通信的中间软件抽象层，一组接口，把复杂的TCP/IP协议族隐藏在Socket接口后面

#短连接
## 连接->传输数据->关闭连接
    HTTP是无状态的，浏览器和服务器每进行一次HTTP操作，就建立一次连接，但任务结束就中断连接。
    也可以这样说：短连接是指SOCKET连接后发送后接收完数据后马上断开连接。
#长连接
## 连接->传输数据->保持连接 -> 传输数据-> 。。。 ->关闭连接。
    长连接指建立SOCKET连接后不管是否使用都保持连接，但安全性较差。
http的长连接：
    HTTP也可以建立长连接的，使用Connection:keep-alive，HTTP 1.1默认进行持久连接。HTTP1.1和HTTP1.0相比较而言，最大的区别就是增加了持久连接支持(貌
 似最新的 http1.0 可以显示的指定 keep-alive),但还是无状态的，或者说是不可以信任的。

 # 参考文章
 ## http://www.jianshu.com/p/584707554ed7
