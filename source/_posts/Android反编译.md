---
title: Android 反编译
date: 2016-12-28 20:20:20
categories:
- 开发
- Android
tags:
- Android
---

修改完善之前写的反编译，更新了自己最近使用的工具

<!--more-->

## Apktool
[apktool](https://ibotpeaches.github.io/Apktool/)用来反编译资源文件
### 下载配置
```
mkdir apktool
cd apktool
wget https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool
wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.2.1.jar
mv apktool_2.2.1.jar apktool.jar
# 将apktool加入系统变量或者修改
sudo cp apktool /usr/local/bin
sudo cp apktool.jar /usr/local/bin
sudo chmod a+x apktool
sudo chmod a+x apktool.jar
```
### 使用
```
# 反编译除资源文件
apktool d HtcContacts.apk

# 重新生产apk
apktool b bar -o new_bar.apk

-f 如果目标文件夹已存在，强制删除现有文件夹
-o 指定反编译的目标文件夹的名称（默认会将文件输出到以Apk文件名命名的文件夹中）
-s 保留classes.dex文件（默认会将dex文件解码成smali文件）
-r 保留resources.arsc文件（默认会将resources.arsc解码成具体的资源文件）
```

## Enjarify
[Enjarify](https://github.com/google/enjarify)反编译dex文件。dex2jar的替代者
enjarify 是python3的程序，注意切换python环境到python3
```
git clone git@github.com:google/enjarify.git
cd enjarify
ln -s "$PWD/enjarify.sh" ~/bin/enjarify
enjarify test.apk
```
这里已经将apk中的dex文件转换成jar包,然后就是查看了
## JD-GUI
[jd-gui](http://jd.benow.ca/)查看jar内容的工具,可以下载jar包，双击运行，然后打开enjarify生产的jar包查看代码
或者下载对应系统的版本运行查看

## 完整流程
反编译打包流程
- apktool反编译修改资源文件
- enjarify或者dex2jar反编译dex，查看java代码,在上面文件中修改对应smail
- apktool 重新生成apk
- 签名
### 签名
```
# 生成签名文件（jdk自带）
keytool -genkey -keystore test.keystore  -alias test -keyalg RSA -validity 10000
# 签名
jarsigner -verbose -keystore test.keystore -signedjar -signed.apk unsigned.apk 'test.keystore'
```

## ClassyShark
[ClassyShark](https://github.com/google/android-classyshark)自动反编译，一键查看
## APK Analyzer
Android Studio2.2 版本加入新功能,一键查看反编译文件


## apk 文件结构

 META-INF 签名文件
 res 资源文件
 AndroidManifest.xml 主配置文件
 classes.dex java 编译产生的字节码文件
 resources.arsc 资源索引
 其他文件 assets lib 等

## 打包流程
APK是Android Package的缩写，实际上APK就是一个zip压缩包，使用zip解压软件直接就能对其进行解压，解压后会发现就是由各种资源文件、一或多个dex文件（odex过的apk除外）、AndroidManifest.xml、resources.arsc以及其他一些文件组成的。

apk 构建流程：https://developer.android.com/studio/build/index.html#detailed-build
- 1 build-tools\aapt aapt编译资源文件
- 2 build-tools\aidl aidl生成java 代码
- 3 java 源码编译(build/intermediates/classes 输出的class文件)
- 4 build-tools\lib\dx.jar 将生成的class文件转换为Dalvik字节码
- 5 生产apk
- 6 签名
- 7 buildtools\zipalign 对签名后的apk文件进行对齐处理，使apk中所有资源文件距离文件起始偏移为4字节的整数倍，从而在通过内存映射访问apk文件时会更快
