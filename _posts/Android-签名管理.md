title: Android 签名管理
date: 2016-03-15 16:51:48
tags:
- Android
- 签名
- 打包
---
# 采用jdk生成签名
## 
```bash
keytool -genkey -alias android.keystore -keyalg RSA -validity 20000 -keystore android.keystore 
```
# 查看签名信息
##
```bash
keytool -list -v -keystore debug.keystore 
```
# 给项目签名
## 可以在项目文件下新建keystoere.properties,内容写上
```bash
keystore=sw.keystore
storePassword=123456
keyAlias=sw.keystore
keyPassword=123456
```
把事先生成的签名文件复制在根目录下。
#Gradle 脚本
##
```bash
 java.io.File signFile=rootProject.file("keystoere.properties")
    if(signFile.exists()){
        System.println("file exist");
        Properties properties=new Properties();
        properties.load(new FileInputStream(signFile));
        signingConfigs{
            release{
                storeFile rootProject.file(properties['keystore'])
                storePassword properties['storePassword']
                keyAlias properties['keyAlias']
                keyPassword properties['keyPassword']
                System.println(storeFile);
                System.println(keyAlias);
                System.println(keyPassword);
            }
        }
    }else{
        System.println("file not exist");
    }

```
注意：写脚本时候，变量要定义在被引用之前；
如果出现读取签名文件错误 检查信息是否写错；

#运行脚本打包
##
1. ./gradlew assemRelease 正式包
2. ./gradlew assemDebug 打测试包

参考文章：https://yeungeek.gitbooks.io/gradle-plugin-user-guide/content/signing_configurations.html