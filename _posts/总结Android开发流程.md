title: 总结Android开发流程
date: 2016-08-06 17:35:34
tags:
- Android
- 项目流程
---
#前言
##对于开发App 无论你是在哪个公司，无论你是不是公司的主程，还是说小弟，我觉得吧，你都应该有一套完成开发一个App的流程，当然这里主要是说安卓开发。
#开发环境选择
##很早之前是采用eclipse，自从进了美芽，在前辈的带领下开启了AndroidStudio之旅，越来越觉得AndroidStudio的好处，从此就用这开发了。
#模拟器的选择
##选好的模拟器很重要的，尤其是运行速度，不能卡。Genymotion是好的选择。
#产品开发流程
##正常的互联网开发app的流程大致如下：

-产品规划，定产品方向
=需求调研，产出需求文档
-需求评审，修订需求文档
-产品狗画app线框图提供给射鸡师
-射鸡师根据线框图设计视觉稿
-程序猿根据视觉稿搭建UI框架
-程序猿根据需求文档开发功能
-测试媛编写测试用例，根据排期进行测试
-程序猿修复回归测试反馈的bug，提交beta版
-测试通过，提交给运营喵发布到渠道上线


#定义一套开发规范
##这边规范主要主要是命名规范（项目命名、包命名、变量命名、资源文件命名、http://blog.csdn.net/wwj_748/article/details/42347283）
#代码管理
##一个产品迭代是必须的最好是采用git管理，可以自己搭建gitlab
#架构搭建
##Android应用其实就是简单的MVC－MVP框架，基本的视图与数据分离方式；
#架构设计
##包括接口设计、技术选型、数据层设计、业务层设计、展示层设计
#接口设计
##1.安全机制  RESTful，设计Token用户用密码登录成功后，服务器返回token给客户端；
客户端将token保存在本地，发起后续的相关请求时，将token发回给服务器；
服务器检查token的有效性，有效则返回数据，若无效，分两种情况：
token错误，这时需要用户重新登录，获取正确的token
token过期，这时客户端需要再发起一次认证请求，获取新的token
#接口数据设计
##JSON数据格式
Number：整数或浮点数
String：字符串
Boolean：true 或 false
Array：数组包含在方括号[]中
Object：对象包含在大括号{}中
Null：空类型
#接口版本设计/v2 /v1
#技术选型
##主要是H5/Native的抉择

#轮子的选择
##UI框架（比如下拉刷新PullToRefresh、侧滑菜单Slidingmenu）
网络请求库（比如okhtttp、AndroidAsyncHttp、Volley）
数据操作库（比如GreenDao、Ormlite）
图片缓存框架（比如Universal-Imageloader）
数据解析库（比如Gson)

#第三方集成
#云测
#混淆
#打包
#APP壳


参考：http://www.jianshu.com/p/42c249168275
http://keeganlee.me/post/architecture/20160222


