---
title: Android启动模式
date: 2016-08-18 10:42:14
categories:
- 开发
- Android
tags:
- Android
---

更好的运用启动模式，可以提高应用交互，简单记录4种启动模式

Task： task 是Android应用运行过程中，Activity的集合，用于控制程序跳转和返回


## standard
默认启动模式，每次Intent请求都会产生，一个新的Activity实例。
可以通过任务管理器查看栈情况

使用场景：standard这种启动模式适合于撰写邮件Activity或者社交网络消息发布Activity。如果你想为每一个intent创建一个Activity处理，那么就是用standard这种模式。

## singleTop
singleTop其实和standard几乎一样，使用singleTop的Activity也可以创建很多个实例。唯一不同的就是，如果调用的目标Activity已经位于调用者的Task的栈顶，则不创建新实例，而是使用当前的这个Activity实例，并调用这个实例的onNewIntent方法

使用场景：搜索功能，多次搜索始终使用一个页面，一次回退既可以回到飞搜索页面

## singleTask
singleTask启动模式的Activity在系统中只会存在一个实例。如果这个实例已经存在，intent就会通过onNewIntent传递到这个Activity。否则新的Activity实例被创建。

singleTask启动模式启动Activity时，首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈

如果不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去
如果存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例
              如果存在实例，则将它上面的Activity实例都出栈，然后回调启动的Activity实例的onNewIntent方法
              如果不存在该实例，则新建Activity，并入栈


## singleInstance
这个模式和singleTask差不多，因为他们在系统中都只有一份实例。唯一不同的就是存放singleInstance Activity实例的Task只能存放一个该模式的Activity实例。

从singleInstance Activity实例启动另一个Activity，那么这个Activity实例会放入其他的Task中。同理，如果singleInstance Activity被别的Activity启动，它也会放入不同于调用者的Task中。

## taskAffinity
taskAffinity 默认为包名
taskAffinity属性不对standard和singleTop模式有任何影响，即时你指定了该属性为其他不同的值，这两种启动模式下不会创建新的task（如果不指定即默认值，即包名）
