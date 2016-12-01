---
title: Android动画总结
date: 2015-05-12 20:20:20
categories:
- 开发
- Android
tags:
- Android
---

android 动画基本使用总结

<!--more-->

## 补间动画
Android的animation包括4中类型：
xml中：alpha 透明， scale 尺寸伸缩， translate 平移， rotale 旋转
代码中：AlphaAnimation 透明，ScaleAnimation 尺寸伸缩，TranslateAnimation 平移，RotateAnimation 旋转

- 1. 新建anim文件夹
- 2. 新建xml文件

```
xml
 <alpha
    android:fromAlpha="1.0"
    android:toAlpha="0.0"
    android:duration="300" />
    <!-- 透明度控制动画效果 alpha
        浮点型值：
            fromAlpha 属性为动画起始时透明度
            toAlpha   属性为动画结束时透明度
            说明:
                0.0表示完全透明
                1.0表示完全不透明
            以上值取0.0-1.0之间的float数据类型的数字

        长整型值：
            duration  属性为动画持续时间
            说明:
                时间以毫秒为单位-->
    <scale
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"
        android:fromXScale="0.0"
        android:toXScale="1.4"
        android:fromYScale="0.0"
        android:toYScale="1.4"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fillAfter="false"
        android:duration="700" />

        <!-- 尺寸伸缩动画效果 scale
       属性：interpolator 指定一个动画的插入器
        有三种动画插入器:
            accelerate_decelerate_interpolator  加速-减速 动画插入器
            accelerate_interpolator        	加速-动画插入器
            decelerate_interpolator        	减速- 动画插入器
        其他的属于特定的动画效果
      浮点型值：

            fromXScale 属性为动画起始时 X坐标上的伸缩尺寸
            toXScale   属性为动画结束时 X坐标上的伸缩尺寸

            fromYScale 属性为动画起始时Y坐标上的伸缩尺寸
            toYScale   属性为动画结束时Y坐标上的伸缩尺寸

            说明:
                 以上四种属性值

                    0.0表示收缩到没有
                    1.0表示正常无伸缩
                    值小于1.0表示收缩
                    值大于1.0表示放大

            pivotX     属性为动画相对于物件的X坐标的开始位置
            pivotY     属性为动画相对于物件的Y坐标的开始位置

            说明:
                    以上两个属性值 从0%-100%中取值
                    50%为物件的X或Y方向坐标上的中点位置

        长整型值：
            duration  属性为动画持续时间
            说明:   时间以毫秒为单位

        布尔型值:
            fillAfter 属性 当设置为true ，该动画转化在动画结束后被应用
-->
 <translate
        android:fromXDelta="30"
        android:toXDelta="-80"
        android:fromYDelta="30"
        android:toYDelta="300"
        android:duration="2000"
        />
    <!-- translate 位置转移动画效果
            整型值:
                fromXDelta 属性为动画起始时 X坐标上的位置
                toXDelta   属性为动画结束时 X坐标上的位置
                fromYDelta 属性为动画起始时 Y坐标上的位置
                toYDelta   属性为动画结束时 Y坐标上的位置
                注意:
                         没有指定fromXType toXType fromYType toYType 时候，
                         默认是以自己为相对参照物
            长整型值：
                duration  属性为动画持续时间
                说明:   时间以毫秒为单位
    -->

    <rotate
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"
        android:fromDegrees="0"
        android:toDegrees="+350"
        android:pivotX="50%"
        android:pivotY="50%"
        android:duration="3000" />
    <!-- rotate 旋转动画效果
           属性：interpolator 指定一个动画的插入器
                 有三种动画插入器:
                    accelerate_decelerate_interpolator   	加速-减速 动画插入器
                    accelerate_interpolator               	加速-动画插入器
                    decelerate_interpolator               	减速- 动画插入器
                 其他的属于特定的动画效果

           浮点数型值:
                fromDegrees 属性为动画起始时物件的角度
                toDegrees   属性为动画结束时物件旋转的角度 可以大于360度


                说明:
                         当角度为负数——表示逆时针旋转
                         当角度为正数——表示顺时针旋转
                         (负数from——to正数:顺时针旋转)
                         (负数from——to负数:逆时针旋转)
                         (正数from——to正数:顺时针旋转)
                         (正数from——to负数:逆时针旋转)

                pivotX     属性为动画相对于物件的X坐标的开始位置
                pivotY     属性为动画相对于物件的Y坐标的开始位置

                说明:        以上两个属性值 从0%-100%中取值
                             50%为物件的X或Y方向坐标上的中点位置

            长整型值：
                duration  属性为动画持续时间
                说明:       时间以毫秒为单位
    -->
```

```
java
myAnimation= AnimationUtils.loadAnimation(this, R.anim.my_anim);
//使用AnimationUtils类的静态方法loadAnimation()来加载XML中的动画XML文件
```

## 帧动画

image_anim.xml
```xml
<?xml version="1.0" encoding="utf-8"?>  
        <animation-list xmlns:android="http://schemas.android.com/apk/res/android"  
            android:oneshot="false">  
            <item android:drawable="@drawable/icon1" android:duration="200" />  
            <item android:drawable="@drawable/icon2" android:duration="200" />  
            <item android:drawable="@drawable/icon3" android:duration="200" />  
        </animation-list>
```
调用代码
```
ImageView imageView = (ImageView) findViewById(R.id.image_view);  
imageView.setBackgroundResource(R.drawable.image_anim);  
AnimationDrawable imageAnimation = (AnimationDrawable) imageView.getBackground();  
imageAnimation.start();
```


## 属性动画

res/animator目录下创建文件
属性动画支持的 Tag 有
```
ValueAnimator - <animator>
ObjectAnimator - <objectAnimator>
AnimatorSet - <set>、
```
xml
```
xml
<set android:ordering="sequentially">
    <set>
        <objectAnimator
            android:propertyName="x"
            android:duration="500"
            android:valueTo="400"
            android:valueType="intType"/>
        <objectAnimator
            android:propertyName="y"
            android:duration="500"
            android:valueTo="300"
            android:valueType="intType"/>
    </set>
    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueTo="1f"/>
</set>
```

使用代码

```
AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext,
    R.anim.property_animator);
set.setTarget(myObject);
set.start();
```

 - set
动画集合节点，有一个属性 ordering，表示它的子动画启动方式是先后有序的还是同时。
属性
sequentially：动画按照先后顺序
together (default) ：动画同时启动
 - objectAnimator
一个对象的一个属性，相应的 Java 类为:ObjectAnimator

属性说明

 - android:propertyName
String 类型，必须要设定的值，代表要执行动画的属性，通过名字引用，比如你可以指定了一个 View 的"alpha" 或者 "backgroundColor"，这个 objectAnimator 元素没有暴露 target 属性，因此不能够在 XML 中执行一个动画，必须通过调用loadAnimator() 填充你的 XML 动画资源，并且调用setTarget() 应用到拥有这个属性的目标对象上。
 - android:valueTo
Float、int 或者 color，也是必须值，表明了动画结束的点，颜色由 6 位十六进制的数字表示。
 - android:valueFrom
相对应 valueTo，动画的起始点，如果没有指定，系统会通过属性身上的 get 方法获取，颜色也是 6 位十六进制的数字表示。
 - android:duration
动画的时长，int 类型，以毫秒为单位，默认为 300 毫秒。
 - android:startOffset
动画延迟的时间，从调用 start 方法后开始计算，int 型，毫秒为单位，
 - android:repeatCount
一个动画的重复次数，int 型，”-1“表示无限循环，”1“表示动画在第一次执行完成后重复执行一次，也就是两次，默认为 0，不重复执行。
 - android:repeatMode
重复模式：int 型，当一个动画执行完的时候应该如何处理。该值必须是正数或者是 -1，
“reverse”
会使得按照动画向相反的方向执行，可实现类似钟摆效果。
“repeat”
会使得动画每次都从头开始循环。
 - android:valueType
关键参数，如果该 value 是一个颜色，那么就不需要指定，因为动画框架会自动的处理颜色值。有 intType 和 floatType 两种：分别说明动画值为 int 和 float 型。

animator

在一个特定的时间里执行一个动画。相对应的是 ValueAnimator.所有的属性和一样 android:valueTo
android:valueFrom
android:duration
android:startOffset
android:repeatCount
android:repeatMode
android:valueType
Value Description
floatType (default)

为了执行该动画，必须在代码中将该动画资源文件填充为一个 AnimationSet 对象，然后在执行动画前，为目标对象设置所有的动画集合。
简便的方法就是通过 setTarget 方法为目标对象设置动画集合，代码如下：
```
AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext,  
   R.anim.property_animator);  
set.setTarget(myObject);  
set.start();
```

参考链接：[http://b.codekk.com/blogs/detail/559623d8d6459ae793499787][1]

[1]: http://b.codekk.com/blogs/detail/559623d8d6459ae793499787
