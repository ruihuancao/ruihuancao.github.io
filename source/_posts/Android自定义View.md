---
title: Android自定义View
date: 2015-08-18 12:08:39
categories:
- 开发
- Android
tags:
- Android
- TODO
---

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
