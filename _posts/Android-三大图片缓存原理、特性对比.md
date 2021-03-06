title: Android Glide/Fresco/Picasso
date: 2017-01-04 16:36:28
tags:
---
#基本信息
Picasso 是 Square 开源的项目，且他的主导者是 JakeWharton，所以广为人知。
Glide 是 Google 员工的开源项目，被一些 Google App 使用，在去年的 Google I/O 上被推荐，不过目前国内资料不多。
 
Fresco 是 Facebook 在今年上半年开源的图片缓存，主要特点包括：
(1) 两个内存缓存加上 Native 缓存构成了三级缓存
(2) 支持流式，可以类似网页上模糊渐进式显示图片
(3) 对多帧动画图片支持更好，如 Gif、WebP

#基本概念
在正式对比前，先了解几个图片缓存通用的概念：
(1) RequestManager：请求生成和管理模块
(2) Engine：引擎部分，负责创建任务(获取数据)，并调度执行
(3) GetDataInterface：数据获取接口，负责从各个数据源获取数据。
比如 MemoryCache 从内存缓存获取数据、DiskCache 从本地缓存获取数据，下载器从网络获取数据等。
(4) Displayer：资源(图片)显示器，用于显示或操作资源。
比如 ImageView，这几个图片缓存都不仅仅支持 ImageView，同时支持其他 View 以及虚拟的 Displayer 概念。
(5) Processor 资源(图片)处理器
负责处理资源，比如旋转、压缩、截取等。
以上概念的称呼在不同图片缓存中可能不同，比如 Displayer 在 ImageLoader 中叫做 ImageAware，在 Picasso 和 Glide 中叫做 Target。

#共同优点

1. 使用简单
都可以通过一句代码可实现图片获取和显示。
2. 可配置度高，自适应程度高
图片缓存的下载器(重试机制)、解码器、显示器、处理器、内存缓存、本地缓存、线程池、缓存算法等大都可轻松配置。
自适应程度高，根据系统性能初始化缓存配置、系统信息变更后动态调整策略。
比如根据 CPU 核数确定最大并发数，根据可用内存确定内存缓存大小，网络状态变化时调整最大并发数等。
3. 多级缓存
都至少有两级缓存、提高图片加载速度。
4. 支持多种数据源
支持多种数据源，网络、本地、资源、Assets 等
5. 支持多种 Displayer
不仅仅支持 ImageView，同时支持其他 View 以及虚拟的 Displayer 概念。
其他小的共同点包括支持动画、支持 transform 处理、获取 EXIF 信息等。

#Picasso 设计及优点

Dispatcher 负责分发和处理 Action，包括提交、暂停、继续、取消、网络状态变化、重试等等。
简单的讲就是 Picasso 收到加载及显示图片的任务，创建 Request 并将它交给 Dispatcher，Dispatcher 分发任务到具体 RequestHandler，任务通过 MemoryCache 及 Handler(数据获取接口) 获取图片，图片获取成功后通过 PicassoDrawable 显示到 Target 中。
需要注意的是上面 Data 的 File system 部分，Picasso 没有自定义本地缓存的接口，默认使用 http 的本地缓存，API 9 以上使用 okhttp，以下使用 Urlconnection，所以如果需要自定义本地缓存就需要重定义 Downloader。
 
2. Picasso 优点
(1) 自带统计监控功能
支持图片缓存使用的监控，包括缓存命中率、已使用内存大小、节省的流量等。
(2) 支持优先级处理
每次任务调度前会选择优先级高的任务，比如 App 页面中 Banner 的优先级高于 Icon 时就很适用。
(3) 支持延迟到图片尺寸计算完成加载
(4) 支持飞行模式、并发线程数根据网络类型而变
手机切换到飞行模式或网络类型变换时会自动调整线程池最大并发数，比如 wifi 最大并发为 4， 4g 为 3，3g 为 2。
这里 Picasso 根据网络类型来决定最大并发数，而不是 CPU 核数。
(5) “无”本地缓存
无”本地缓存，不是说没有本地缓存，而是 Picasso 自己没有实现，交给了 Square 的另外一个网络库 okhttp 去实现，这样的好处是可以通过请求 Response Header 中的 Cache-Control 及 Expired 控制图片的过期时间。
 
六、Glide 设计及优点

1. 总体设计及流程
上面是 Glide 的总体设计图。整个库分为 RequestManager(请求管理器)，Engine(数据获取引擎)、 Fetcher(数据获取器)、MemoryCache(内存缓存)、DiskLRUCache、Transformation(图片处理)、Encoder(本地缓存存储)、Registry(图片类型及解析器配置)、Target(目标) 等模块。
简单的讲就是 Glide 收到加载及显示资源的任务，创建 Request 并将它交给 RequestManager，Request 启动 Engine 去数据源获取资源(通过 Fetcher )，获取到后 Transformation 处理后交给 Target。
Glide 依赖于 DiskLRUCache、GifDecoder 等开源库去完成本地缓存和 Gif 图片解码工作。

2. Glide 优点
(1) 图片缓存->媒体缓存
Glide 不仅是一个图片缓存，它支持 Gif、WebP、缩略图。甚至是 Video，所以更该当做一个媒体缓存。
(2) 支持优先级处理
(3) 与 Activity/Fragment 生命周期一致，支持 trimMemory
Glide 对每个 context 都保持一个 RequestManager，通过 FragmentTransaction 保持与 Activity/Fragment 生命周期一致，并且有对应的 trimMemory 接口实现可供调用。
(4) 支持 okhttp、Volley
Glide 默认通过 UrlConnection 获取数据，可以配合 okhttp 或是 Volley 使用。实际 ImageLoader、Picasso 也都支持 okhttp、Volley。
(5) 内存友好
① Glide 的内存缓存有个 active 的设计
从内存缓存中取数据时，不像一般的实现用 get，而是用 remove，再将这个缓存数据放到一个 value 为软引用的 activeResources map 中，并计数引用数，在图片加载完成后进行判断，如果引用计数为空则回收掉。
② 内存缓存更小图片
Glide 以 url、view_width、view_height、屏幕的分辨率等做为联合 key，将处理后的图片缓存在内存缓存中，而不是原始图片以节省大小
③ 与 Activity/Fragment 生命周期一致，支持 trimMemory
④ 图片默认使用默认 RGB_565 而不是 ARGB_888
虽然清晰度差些，但图片更小，也可配置到 ARGB_888。
其他：Glide 可以通过 signature 或不使用本地缓存支持 url 过期