title: Android H5 采用Gradle管理离线包
date: 2016-06-06 17:35:25
tags:
- Android
- h5
- gradle
---
# H5离线访问
## 离线访问，就是先将H5放在本地，提高加载速度
# H5要有一个版本号（线上版本号）
## 为了实现同一资源的线上和离线访问，Native需要对H5的静态资源请求进行拦截判断，将静态资源“映射”到sd卡资源，即实现一个处理H5资源的本地路由
# H5 Gradle

```java
import com.squareup.okhttp.OkHttpClient
import com.squareup.okhttp.Request
import com.squareup.okhttp.Response
import groovy.json.JsonSlurper

import java.util.zip.ZipEntry
import java.util.zip.ZipInputStream

buildscript {
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath 'com.squareup.okhttp:okhttp:2.4.0'
    }
}

task updateH5 << {
    File tmp = new File("Meiya/build/tmp/h5")
    File assets_h5 = new File("Meiya/assets/h5")

    def deleteFile;
    deleteFile = { File file ->
        if (!file.exists()) return
        if (file.isFile()) {
            file.delete();
        } else {
            file.listFiles().each {
                deleteFile(it)
            }
            file.delete()
        }
    }
    deleteFile(tmp)
    deleteFile(assets_h5)

    assets_h5.mkdirs()
    tmp.mkdirs()

    //   def SIGNATURE = "knOV3ucpaNHZ6wpiXIkyK3acasNWz8bbskYxMkPu5zNImXKssgFkgpn020D5"
    // def URL_HOST_MEIYA = "api.meiyaapp.com"

    //  def URL_CONFIGURATION = "http://${URL_HOST_MEIYA}/v1/configurations?signature=${SIGNATURE}"

    OkHttpClient client = new OkHttpClient();
    JsonSlurper slurper = new JsonSlurper()

    //先获取最新的h5版本
//    def h5_version = slurper.parseText(
//            httpGetString(client, URL_CONFIGURATION)).data.h5_version
    def h5_version = project.H5_VERSION
    println "h5 version: ${h5_version}"

    //再获取所有的zip包下载地址。
    def h5_package_map_string = httpGetString(client, "http://st0.meiyaapp.com/app/${h5_version}/H5PackageMap.json");
    //H5PackageMap.json --> Meiya/assets/h5/H5PackageMap.json
    writeToLocalH5PackageMapFile(h5_package_map_string, assets_h5)
    def h5_package_map = slurper.parseText(h5_package_map_string)

    //下载所有的zip包。

    h5_package_map.H5PackageList.values().each { value ->
        //http://st0.meiyaapp.com/app/a0d6fce8f08fb3c5617e.zip ---> Meiya/build/tmp/h5/a0d6fce8f08fb3c5617e.zip
        httpDownloadFile(client, value, new File(tmp, parseFileNameFromUrl(value)))
    }
    //解压
    tmp.listFiles().each { zip ->
        if (zip.getName().endsWith("zip")) {
            //Meiya/build/tmp/h5/a0d6fce8f08fb3c5617e.zip --> a0d6fce8f08fb3c5617e/*
            unZipIt(zip.getAbsolutePath(), tmp.getAbsolutePath())

            String md5 = removeExtFromFileName(zip.getName())
            File src = new File(tmp, md5)
            File asset_dest = new File(assets_h5, md5)
            if (!src.exists()) {
                throw new Exception(src.getAbsolutePath() + " is not exited")
            }
            if (!src.renameTo(asset_dest)) {
                throw new Exception(src.getAbsolutePath() + " rename failed.")
            }
            println "${src.getAbsolutePath()} --> ${asset_dest.getAbsolutePath()}"
        }
    }
    "git status".execute()
    "git add . --all".execute()
    println "git commit -m \"Update h5 to $h5_version\""
}

/**
 * Unzip it
 * @param zipFile input zip file
 * @param output zip file output folder
 */
private void unZipIt(String zipFile, String outputFolder) {


    byte[] buffer = new byte[1024];

    try {
        //create output directory is not exists
        File folder = new File(outputFolder);
        if (!folder.exists()) {
            folder.mkdir();
        }

        //get the zip file content
        ZipInputStream zis =
                new ZipInputStream(new FileInputStream(zipFile));
        //get the zipped file list entry
        ZipEntry ze = zis.getNextEntry();

        while (ze != null) {
            String fileName = ze.getName();
            File newFile = new File(outputFolder + File.separator + fileName);

            if (ze.isDirectory()) {
                newFile.mkdirs()
                ze = zis.getNextEntry();
                continue;
            }

            //  System.out.println("file unzip : " + newFile.getName());

            //create all non exists folders
            //else you will hit FileNotFoundException for compressed folder
            new File(newFile.getParent()).mkdirs();

            FileOutputStream fos = new FileOutputStream(newFile);

            int len;
            while ((len = zis.read(buffer)) > 0) {
                fos.write(buffer, 0, len);
            }

            fos.close();
            ze = zis.getNextEntry();
        }

        zis.closeEntry();
        zis.close();

        System.out.println("Done");

    } catch (IOException ex) {
        ex.printStackTrace();
    }
}

private String writeToLocalH5PackageMapFile(String content, File assets_h5) {
    File local_h5_package_map_file = new File(assets_h5, "H5PackageMap.json")
    local_h5_package_map_file.delete()

    local_h5_package_map_file << content
}

private String parseFileNameFromUrl(String url) {
    if (url.contains("?")) {
        url = url.substring(0, url.lastIndexOf("?"));
    }
    return url.substring(url.lastIndexOf('/') + 1, url.length());
}

private String removeExtFromFileName(String fileName) {
    return fileName.substring(0, fileName.lastIndexOf('.'));
}

private String httpGetString(OkHttpClient client, String url) {
    Request req = new Request.Builder()
            .url(url)
            .header("Accept", "application/json")
            .build();
    Response response = client.newCall(req).execute()

    return response.body().string()
}

private void httpDownloadFile(OkHttpClient client, String url, File dest) {
    if (dest.exists()) return

    Request req = new Request.Builder()
            .url(url)
            .build();
    Response response = client.newCall(req).execute()

    def out = dest.newOutputStream()
    out << response.body().byteStream()
    out.flush()
    out.close()
}

```

#参考文章
## https://segmentfault.com/a/1190000004263182
