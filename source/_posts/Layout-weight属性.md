---
title: Layout_weight属性
date: 2014-08-18 09:50:24
categories:
- Android
tags:
- Android
---

这个本来应该删了，因为没什么用，当时不理解这个是因为没有看View绘制流出的源码，留着看看以前有多傻。。。
<!--more-->

## layout_weight例子1

```
<LinearLayout
       android:orientation="horizontal"
       android:layout_width="match_parent"
       android:layout_height="wrap_content">
       <Button
           android:layout_weight="1"
           android:padding="8dp"
           android:text="Button1"
           android:layout_width="match_parent"
           android:layout_height="wrap_content" />
       <Button
           android:layout_weight="2"
           android:padding="8dp"
           android:text="Button2"
           android:layout_width="match_parent"
           android:layout_height="wrap_content" />
   </LinearLayout>
```
以上： Button1 占2/3  Button2 占1/3

## layout_weight例子2
```
<LinearLayout
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <Button
        android:layout_weight="1"
        android:padding="8dp"
        android:text="Button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
    <Button
        android:layout_weight="2"
        android:padding="8dp"
        android:text="Button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```
以上： Button1 占1/3 Button2 占2/3

不同在于Button的layout_width属性：
当android:layout_width="match_parent"，此时weight所在view，尽可能最大
当android:layout_width="wrap_content"，此时weight所在view，尽可能最小

## layout_weight例子3
```
<LinearLayout
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <Button
        android:layout_weight="1"
        android:padding="8dp"
        android:text="Button1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <Button
        android:layout_weight="2000"
        android:padding="8dp"
        android:text="Button2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```
Button1占满父布局
对比1: Button2 weight为2000，意味着Button1很大，最大也是占满父布局。

## layout_weight例子4
```
<LinearLayout
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <Button
        android:layout_weight="1"
        android:padding="8dp"
        android:text="Button1"
        android:layout_width="2dp"
        android:layout_height="wrap_content" />
    <Button
        android:layout_weight="2"
        android:padding="8dp"
        android:text="Button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```
Button1 并没有2dp，而是显示内容的最小宽度即warp_content
对比2: layout_width 为2dp Button1 很小，最小宽度也是warp_content


总结：
当android:layout_width="match_parent"，此时weight所在view，尽可能最大，最大为父布局
当android:layout_width="wrap_content"，此时weight所在view，尽可能最小，最小为内容包裹大小
