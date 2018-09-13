---
title: Ubuntu下adb找不到手机
date: 2015-03-13 20:31:46
categories:
- Android
tags:
- Ubuntu
---
Ubuntu下连接手机，adb找不到，好奇怪，想起源码编译的时候，需要加Usb Access，查了下，果然如此呀

<!--more-->
解决办法：
- 1 终端运行lsusb查看设备Id.怎么看呢？插上手机看一下，拔掉手机看一下，对比就知道那个是无法显示的设备.
 id的值是这样的idVendor:idProduct
- 2 将USB设备的ID加入到UDEV规则中
```
sudo gedit  /etc/udev/rules.d/51-android.rules  

sudo udevadm control --reload-rules
```
复制一行 修改里面的idVendor和idProduct，加进去
- 3 重启adb就可以了
