---
title: Android自定义View
date: 2015-09-15 12:08:39
categories:
- 开发
- Android
tags:
- Android
- TODO
---
Android 自定义View大概是开发中必不可少的，自定义View并不是很复杂，为什么感觉很炫的效果，感觉实现没思路呢，首先搞清楚基本原理和流程，剩下的就是动画和绘图变化的操作了。看了网上很多文章总结，后来发现还不如自己看一遍源码，别人写的东西，大多是有自己的理解在里面，如果要自己理解还是看源码直接，主要参照View类和ViewGroup类，事件分发也是一样。

<!--more-->


## 自定义view不同机型适配


## View工作流程
View主要流程：measure layout draw

### measure流程
View measure : measure() 是个final方法，其中调用onMeasure()进行测量

View尺寸由父元素的测量模式和View的LayoutParams决定

MeasureSpec.UNSPECIFIED: 父容器不限制View大小，要多大给多大，一般用在系统内部。
MeasureSpec.AT_MOST: 父容器指定一个可用的大小，View最大不能超过该值，具体多大取决于View对onMeasure()的实现。对应于LayoutParams的wrap_parent.
MeasureSpec.EXACITY: 父容器已经准确检测到View的数值大小，最终大小就是SpecSize。对应于LayoutParams的match_parent和具体的数值

measure过程分两部分View measure 和ViewGroup Measure
如果一个View只是一个View，通过measure就可以完成
如果一个View是ViewGroup，需要首先完成对View树便利，先完成子View的measure，最后完成自身的measure

View measure 由measure方法完成，其中调用onMeasure()方法
```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```
getSuggestedMinimumHeight取值：如果View没有背景取minHeight值，有背景则取背景和minHeight的最大值
