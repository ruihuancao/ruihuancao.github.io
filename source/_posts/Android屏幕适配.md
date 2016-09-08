---
title: Android屏幕适配
date: 2016-08-18 12:05:57
categories:
- 开发
- Android
tags:
- Android
- TODO
---

Android不同设备分辨率做UI适配，可以获得使用户获得更好的体验


<!--more-->

## 概念

屏幕尺寸是指屏幕对角线的长度。单位是英寸，1英寸=2.54厘米

屏幕分辨率是指在横纵向上的像素点数，单位是px，1px=1像素点，一般是纵向像素横向像素，如1280×720

屏幕像素密度是指每英寸上的像素点数，单位是dpi，即“dot per inch”的缩写，像素密度和屏幕尺寸和屏幕分辨率有关

dip：Density Independent Pixels（密度无关像素）的缩写。以160dpi为基准，1dp=1px
dp：同dip
dpi：屏幕像素密度的单位，“dot per inch”的缩写

px：像素，物理上的绝对单位
sp：Scale-Independent Pixels的缩写，可以根据文字大小首选项自动进行缩放。

例子：
A设备像素密度：160dpi
B设备像素密度：240dpi
应用画一条线：宽为200dp，像素为？
像素＝（dpi／160）＊dp
A:(160/160)＊200=200px
B:(240/160)＊200=300px

## mdpi hdpi xhdpi xxhdpi xxxhdpi 范围和区分

名称	像素密度范围	图片大小
mdpi	120dp~160dp	48×48px
hdpi	160dp~240dp	72×72px
xhdpi	240dp~320dp	96×96px
xxhdpi	320dp~480dp	144×144px
xxxhdpi	480dp~640dp	192×192px

## 资源文件适配
使用尺寸限定符：res/layout-large/main.xml
使用最小宽度限定符：layout-sw600dp/main.xml
使用屏幕方向限定符：res/values-sw600dp-land/layouts.xml res/values-sw600dp-port/layouts.xml

## 9图
图片每个位置划线作用含义不同
上： 划线区域 横向拉伸
左： 划线区域 纵向拉伸

上，左决定拉伸区域

下： 划线区域为内容横向显示区域
右： 划线区域为内容纵向显示区域

右，下决定内容显示区域
