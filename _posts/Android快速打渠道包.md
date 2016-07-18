title: Android快速打渠道包
date: 2016-07-18 13:49:51
tags:
- Android
- 打包
---
# 修改项目gradle
## 
```bash

buildscript {
    ......
    dependencies{
    // add packer-ng
        classpath 'com.mcxiaoke.gradle:packer-ng:1.0.5'
    }
}
```
# 修改修改moudle级别gradle
## 
```bash
apply plugin: 'packer' 

dependencies {
    // add packer-helper
    compile 'com.mcxiaoke.gradle:packer-helper:1.0.5'
}

```

# 注意：packer-ng 和 packer-helper 的版本号需要保持一致
# 在你的项目根目录中加入渠道列表文件，比如文件名是market.txt，内容是
```bash
Google_Play#play store market
Gradle_Test#test
SomeMarket#some market
```
#打包
## gradle -Pmarket=markets.txt clean apkRelease

#友盟统计
##
最后需要提的一点就是如何让友盟统计知道目前的apk是哪个渠道。首先你需要删除之前的productFlavor或者manifest的占位符的方式的代码，删除AndroidManifest中友盟的渠道Channel的META-Data的配置。
然后在app入口（Application）的onCreate中加入下列代码
```bash
final String market = PackerNg.getMarket(this,"defaul_channel);
AnalyticsConfig.setChannel(market); //AnalyticsConfig是友盟的代码方式设置渠道类
```

