title: AndFix使用说明(android 热修复打补丁)
date: 2016-04-18 22:08:23
tags:
- Android
- 热修复
- AndFix
---
#介绍
## 开源框架阿里巴巴的AndFix,允许APP在不重新发布版本的情况下修复线上的bug。支持Android 2.3 到 6.0。
#使用方式
##添加依赖库
```bash
compile 'com.alipay.euler:andfix:0.4.0@aar'
```
##添加so文件
可以参考https://github.com/zhonghanwen/AndFix-Ndk-Build-ADT
#Application代码
##
```bash
 patchManager = new PatchManager(this);
        patchManager.init(getAppVersion());//current version
        patchManager.loadPatch();
```
---
```bash
try {
            patchManager.addPatch(Environment.getExternalStorageDirectory() + "/v.apatch");
            Log.i("MyApplication","success");
        }catch (Exception ex){
            Log.i("MyApplication",ex.getMessage());
            Log.i("MyApplication",Environment.getExternalStorageDirectory().toString());
        }

public String getAppVersion(){
        String appversion="";
        try {
             appversion = getPackageManager().getPackageInfo(getPackageName(), 0).versionName;
        }catch (PackageManager.NameNotFoundException ex){

        }finally {

        }

        return appversion;
    }
```

#生成补丁文件Patch
##工具 apkpatch

```bash
./apkpatch.sh -f volly2.apk -t volly.apk -k guosen.jks -p 123456 -a lin -e 123456
```
