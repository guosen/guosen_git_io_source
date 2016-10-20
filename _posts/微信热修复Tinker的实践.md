title: 微信热修复Tinker的实践
date: 2016-10-20 15:27:29
tags:
- Android
- TInker
- 热修复
---
#Tinker热修复简介
## Tinker是微信开源出的Android热修复框架，基于MultiDex.

#接入Tinker相关依赖
1.项目的根目录build.gradle:
``bash

dependencies {
        classpath 'com.tencent.tinker:tinker-patch-gradle-plugin:1.7.1'
    }

``
2.项目下的APP目录下的build.gradle:
``bash
compile('com.tencent.tinker:tinker-android-anno:1.7.1')
    //tinker 核心 Android lib
    compile('com.tencent.tinker:tinker-android-lib:1.7.1')

``
3.添加gradle脚本，主要是生成patch包，Tinker是采用插件的形式，你可以参照Tinker－sample－android的做法：
``bash

def bakPath = file("${buildDir}/bakApk/")

/**
 * you can use assembleRelease to build you base apk
 * use tinkerPatchRelease -POLD_APK=  -PAPPLY_MAPPING=  -PAPPLY_RESOURCE= to build patch
 * add apk from the build/bakApk
 */
ext {
    //for some reason, you may want to ignore tinkerBuild, such as instant run debug build?
    tinkerEnabled = true
    //you should bak the following files
    //old apk file to build patch apk
    tinkerOldApkPath = "${bakPath}/Meiya-debug-tt.apk"
    //proguard mapping file to build patch apk
    tinkerApplyMappingPath = "${bakPath}"
    //resource R.txt to build patch apk, must input if there is resource changed
    tinkerApplyResourcePath = "${bakPath}"
}


def getOldApkPath() {
    return hasProperty("OLD_APK") ? OLD_APK : ext.tinkerOldApkPath
}

def getApplyMappingPath() {
    return hasProperty("APPLY_MAPPING") ? APPLY_MAPPING : ext.tinkerApplyMappingPath
}

def getApplyResourceMappingPath() {
    return hasProperty("APPLY_RESOURCE") ? APPLY_RESOURCE : ext.tinkerApplyResourcePath
}

def getTinkerIdValue() {
    return hasProperty("TINKER_ID") ? TINKER_ID : gitSha()
}

def buildWithTinker() {
    return hasProperty("TINKER_ENABLE") ? TINKER_ENABLE : ext.tinkerEnabled
}
def gitSha() {
    try {
        String gitRev = 'git rev-parse --short HEAD'.execute().text.trim()
        if (gitRev == null) {
            throw new GradleException("can't get git rev, you should add git to system path or just input test value, such as 'testTinkerId'")
        }
        return gitRev
    } catch (Exception e) {
        throw new GradleException("can't get git rev, you should add git to system path or just input test value, such as 'testTinkerId'")
    }
}
if (buildWithTinker()) {
    apply plugin: 'com.tencent.tinker.patch'

    tinkerPatch {
        /** 全局信息相关的配置项
         * necessary，default 'null'
         * the old apk path, use to diff with the new apk to build
         * add apk from the build/bakApk
         */
        oldApk = getOldApkPath()
        /**
         如果出现以下的情况，并且ignoreWarning为false，我们将中断编译。因为这些情况可能会导致编译出来的patch包带来风险：
         1. minSdkVersion小于14，但是dexMode的值为"raw";
         2. 新编译的安装包出现新增的四大组件(Activity, BroadcastReceiver...)；
         3. 定义在dex.loader用于加载补丁的类不在main dex中;
         4. 定义在dex.loader用于加载补丁的类出现修改；
         5. resources.arsc改变，但没有使用applyResourceMapping编译
         */
        ignoreWarning = true
        /**
         在运行过程中，我们需要验证基准apk包与补丁包的签名是否一致，我们是否需要为你签名
         */
        useSign = true

        /**
         * Warning, applyMapping will affect the normal android build!
         */
        buildConfig {
            /** 编译相关的配置项
             * optional，default 'null'
             * if we use tinkerPatch to build the patch apk, you'd better to apply the old
             * apk mapping file if minifyEnabled is enable!
             * Warning:
             * you must be careful that it will affect the normal assemble build!
             */
            applyMapping = getApplyMappingPath()
            /**
             可选参数；在编译新的apk时候，我们希望通过旧apk的R.txt文件保持ResId的分配，这样不仅可以减少
             补丁包的大小，同时也避免由于ResId改变导致remote view异常
             */
            applyResourceMapping = getApplyResourceMappingPath()

            /**
             * necessary，default 'null'
             在运行过程中，我们需要验证基准apk包的tinkerId是否等于补丁包的tinkerId。这个
             是决定补丁包能
             运行在哪些基准包上面，一般来说我们可以使用git版本号、versionName等等
             */
            tinkerId = getTinkerIdValue()
        }

        dex {
            /**
             * optional，default 'jar'
             * only can be 'raw' or 'jar'. for raw, we would keep its original format
             * for jar, we would repack dexes with zip format.
             * if you want to support below 14, you must use jar
             * or you want to save rom or check quicker, you can use raw mode also
             */
            dexMode = "jar"
            /**
             * necessary，default '[]'
             * what dexes in apk are expected to deal with tinkerPatch
             * it support * or ? pattern.
             */
            pattern = ["classes*.dex",
                       "assets/secondary-dex-?.jar"]
            /**
             * necessary，default '[]'
             * Warning, it is very very important, loader classes can't change with patch.
             * thus, they will be removed from patch dexes.
             * you must put the following class into main dex.
             * Simply, you should add your own application {@code tinker.sample.android.SampleApplication}
             * own tinkerLoader, and the classes you use in them
             *
             */
            loader = ["com.tencent.tinker.loader.*",
                      "com.meiyaapp.meiya.APP",
                      //use sample, let BaseBuildInfo unchangeable with tinker
                      "tinker.com.sen.mytinkerdemo.BaseBuildInfo"
            ]
        }

        lib {
            /**
             * optional，default '[]'
             * what library in apk are expected to deal with tinkerPatch
             * it support * or ? pattern.
             * for library in assets, we would just recover them in the patch directory
             * you can get them in TinkerLoadResult with Tinker
             */
            pattern = ["lib/armeabi/*.so"]
        }

        res {
            /**
             * optional，default '[]'
             * what resource in apk are expected to deal with tinkerPatch
             * it support * or ? pattern.
             * you must include all your resources in apk here,
             * otherwise, they won't repack in the new apk resources.
             */
            pattern = ["res/*", "assets/*", "resources.arsc", "AndroidManifest.xml"]

            /**
             * optional，default '[]'
             * the resource file exclude patterns, ignore add, delete or modify resource change
             * it support * or ? pattern.
             * Warning, we can only use for files no relative with resources.arsc
             */
            ignoreChange = ["assets/sample_meta.txt"]

            /**
             * default 100kb
             * for modify resource, if it is larger than 'largeModSize'
             * we would like to use bsdiff algorithm to reduce patch file size
             */
            largeModSize = 100
        }

        packageConfig {
            /**
             * optional，default 'TINKER_ID, TINKER_ID_VALUE' 'NEW_TINKER_ID, NEW_TINKER_ID_VALUE'
             * package meta file gen. path is assets/package_meta.txt in patch file
             * you can use securityCheck.getPackageProperties() in your ownPackageCheck method
             * or TinkerLoadResult.getPackageConfigByName
             * we will get the TINKER_ID from the old apk manifest for you automatic,
             * other config files (such as patchMessage below)is not necessary
             */
            configField("patchMessage", "tinker is sample to use")
            /**
             * just a sample case, you can use such as sdkVersion, brand, channel...
             * you can parse it in the SamplePatchListener.
             * Then you can use patch conditional!
             */
            configField("platform", "all")

        }
        //or you can add config filed outside, or get meta value from old apk
        //project.tinkerPatch.packageConfig.configField("test1", project.tinkerPatch.packageConfig.getMetaDataFromOldApk("Test"))
        //project.tinkerPatch.packageConfig.configField("test2", "sample")

        /**
         * if you don't use zipArtifact or path, we just use 7za to try
         */
        sevenZip {
            /**
             * optional，default '7za'
             * the 7zip artifact path, it will use the right 7za with your platform
             */
            zipArtifact = "com.tencent.mm:SevenZip:1.1.10"
            /**
             * optional，default '7za'
             * you can specify the 7za path yourself, it will overwrite the zipArtifact value
             */
//        path = "/usr/local/bin/7za"
        }
    }


}

/**
 * task type, you want to bak
 */
def taskName = "debug"
/**
 * bak apk and mapping
 */

if (buildWithTinker()) {//在Tinker模式下复制bakApk(by shouwang)
    tasks.getByName("assemble${taskName.capitalize()}") {
        it.doLast {
            copy {

                def date = new Date().format("MMdd-HH-mm-ss")
                from "${buildDir}/outputs/apk/${project.getName()}-${taskName}.apk"
                into bakPath
                rename { String fileName ->
                    fileName.replace("${project.getName()}-${taskName}.apk", "${project.getName()}-${taskName}-${date}.apk")
                }

                from "${buildDir}/outputs/mapping/${taskName}/mapping.txt"
                into bakPath
                rename { String fileName ->
                    fileName.replace("mapping.txt", "${project.getName()}-${taskName}-${date}-mapping.txt")
                }

                from "${buildDir}/intermediates/symbols/${taskName}/R.txt"
                into bakPath
                rename { String fileName ->
                    fileName.replace("R.txt", "${project.getName()}-${taskName}-${date}-R.txt")
                }

            }
        }
    }
}

``

＃如何生成补丁，这边有几个步骤
1.修改buildPatch.gradle里面tinkerOldApkPath的值（对应上个apk名称，就是要修复的APK）（运行项目会生成在APP/build/bakApk/ 目录下）
2.要填写TinkerId的值，在gradle.properties里，项目里采用App-VersionName（默认是git版本号）
3.修改buildPatch.gradle里tinkerEnabled的值为true(注：发版本的时候也要用true打包)
4.运行task tinkerPatchDebug.补丁包为patch_signed_7zip.apk，生成目录在App/build/output/tinkerPatch/ 下
5.可以push到手机测试

这里注意一点：Tinker支持对同一基准版本做多次补丁修复，在生成补丁时，oldApk依然是已经发布出去的那个版本。即补丁版本二的oldApk不能是补丁版本一，它应该依然是用户手机上已经安装的基准版本。
``bash
adb push ./Meiya/build/outputs/tinkerPatch/debug/patch_signed_7zip.apk /storage/sdcard0/
``

#关于打包
##我们需要保证tinkerId一定是要唯一性的，这里我们一定要注意，升级可客户端版本，需要更新tinkerId!

#补丁加载
##关于补丁加载，其实主要是项目Applicaion类的改造，改造的话主要是代理：
``bash
public class APP extends TinkerApplication {
 private static APP sInstance;
 public APP() {

     super(
             //tinker标志, 要支持那种类型 默认为ALL
             //dex only, library only, all support
             ShareConstants.TINKER_ENABLE_ALL,
             // 这边最好用完整包名,参数:AppApplication的托管类,Tinker的核心类,是否验证
             // have a binary dependency on your ApplicationLifeCycle class.
             "com.meiyaapp.meiya.MeiyaApplicationLike","com.tencent.tinker.loader.TinkerLoader",false);
     sInstance=this;
 }

 public static APP getInstance() {
     return sInstance;
 }

``
注：Tinker前提是项目的Application类不能引入其他类，否者会导致他们无法被补丁修改。
改造的方法采用委托：这里将全部的实现移到了单独的类里面SampleApplicationLike.java(具体参照项目源码)

