---
title: ReactNative开发
date: 2016-5-3 20:10:20
categories:
- 开发
- ReactNative
tags:
- ReactNative
---

ReactNative
主要思想：组件化，跨平台

<!--more-->

## 重要
  当前问题很多，资料相对较少，出问题解决查找方式
  1 第一个一定是查找官方文档， 不同版本出入很大，官方文档更准确（不要看国内翻译，翻译存在不一定是最新的），文档更新比较慢
  2 google http://stackoverflow.com/  github

## 环境配置及运行hello world
  环境配置参考以下网址：https://facebook.github.io/react-native/docs/getting-started.html
  最新版本0.26.0,maven当前最新是0.20.1 版本不兼容，看清楚自己的js中版本和客户端中版本一致
  要使用0.26.0 参看
  ubuntu14.04 测试配置问题少了很多坑，也详细了很多
  ide选择：sublime,webstorm,Nuclide(官方推荐),安装方式查看对应官方文档

  Nuclide安装：
  - 1 常规安装 https://nuclide.io/docs/editor/setup/#linux
  - 2 源码安装
  ```
    git clone https://github.com/facebook/nuclide.git
    cd nuclide
    npm install
    apm link
  ```

## 调试
  按照官方文档配置:https://facebook.github.io/react-native/docs/running-on-device-android.html#content
  ```
  react-native init newproject //创建新项目
  npm install
  react-native run-android
  adb reverse tcp:8081 tcp:8081 //设置adb端口（5.0以下系统会出错，节省时间，直接换5.0以上系统调试）
  ```
  摇一摇打开调试模式：reload js
```
echo 256 | sudo tee -a /proc/sys/fs/inotify/max_user_instances
echo 32768 | sudo tee -a /proc/sys/fs/inotify/max_queued_events
echo 65536 | sudo tee -a /proc/sys/fs/inotify/max_user_watches
watchman shutdown-server
```
  备注： 查看logcat日志 或者界面日志如果下载bundle 文件地址不对
  在调试模式下设置主机地址及端口号：  电脑ip:8081 (ubuntu 使用ifconfig 查看ip地址)

  Chrome开发调试工具： http://localhost:8081/debugger-ui

## 打包
  react-native bundle 生成bundle文件及所使用的资源文件 打包时加入Android 项目
  react-native bundle  --platform "android"  --entry-file index.android.js --bundle-output app/src/main/assets/index.android.bundle  --assets-dest app/src/main/res/ --dev true

##小结：
   reactnative 通信模型：
   java(native端Android实现)<-->Bridge(reactnative 实现)<---> JavaScript
   这种实现方式在Android 系统实现是Webview (底层实现WebKit 4.4之后换成Chromium，Chromium基本还是webkit变种)
   java <-->(c,c++(bridge<-->webkit))<-->JavaScript

   JavaScript实现的组件打包成bundle文件
   对于Android来说实际上使用的是index.android.bundle文件
   开本地npm 只是为了调试实时刷新
   发布参见打包


## 现有应用中移植入ReactNative
  参见这个文档（注意reactnative版本号,保持一致，版本不同，使用可能不同）
  https://facebook.github.io/react-native/docs/embedded-app-android.html#content
  流程：
  - 1 新建一个Android 项目，并运行
  - 2 引入react native依赖,配置权限
  - 3 使用ReactRootView呈现
  - 4 添加js
