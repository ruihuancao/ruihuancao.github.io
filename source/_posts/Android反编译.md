---
title: Android 反编译纪录
date: 2015-05-28 20:20:20
categories:
- 开发
- Android
tags:
- Android
---

Android 反编译记录，方便查看别人优秀的代码，供学习使用

<!--more-->

## apk 文件结构

 META-INF 签名文件
 res 资源文件
 AndroidManifest.xml 主配置文件
 classes.dex java 编译产生的字节码文件
 resources.arsc 资源索引
 其他文件 assets lib 等

## 工具配置
  apktool http://ibotpeaches.github.io/Apktool/
  jadx https://github.com/skylot/jadx

  jadx
  git clone https://github.com/skylot/jadx.git
  cd jadx
  ./gradlew dist

  添加到环境变量

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
