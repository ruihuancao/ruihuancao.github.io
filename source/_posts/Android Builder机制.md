---
title: Android Builder机制
date: 2017-01-18 12:08:59
categories:
- Android
tags:
- Android
---

IPC：Inter-Process Communication，即进程间通信

AIDL：Android Interface Definition Language，即Android接口定义语言。Client和Service要实现
跨进程通信，必须遵循的接口规范。需要创建.aidl文件，外在表现上和Java中的interface有点类似。
<!-- more -->
Binder：Android进程间通信是通过Binder来实现的。远程Service在Client绑定服务时，会在onBind()的回调中返回一个Binder，当Client调用bindService()与远程Service建立连接成功时，会拿到远程Binder实例，从而使用远程Service提供的服务

Linux系统进程间通信方式：

1. 传统机制。如管道(Pipe)、信号(Signal)和跟踪(Trace)，适用于父子进程或兄弟进程，其中命名管道(Named Pipe)，支持多种类型进程
2. System V IPC机制。如报文队列(Message)、共享内存(Share Memory)和信号量(Semaphore)
3. Socket通信

1、理解远程服务通信机制

通过案例先了解到本地端和服务端跨进程通信，主要就是借助Binder进行功能调用，而在这里主要有两个核心类，一个是Stub类，这个类是继承了Binder类具备了将远程传递的Binder对象转化成本地实际对象asInterface方法即可，同时实现了IXXX接口，需要实现AIDL中的功能方法，还有一个类就是Proxy类，实现了IXXX接口，同时内部保留着远端传递的Binder对象，然后通过这个对象调用远端方法。这里Stub类就是服务端的中间者，而Proxy就是本地端的中间者。

2、系统服务调用流程

通过分析了跨进程通信机制原理之后，再去看看Android系统中在使用一些服务的时候，通过getSystemService方法获取服务对象，其实这内部就是通过跨进程获取到了远端服务的Binder对象，然后转化成系统服务对象给应用调用，而这些系统服务的Binder对象在系统启动的时候服务会自动注册到ServiceManager中。

3、服务大管家ServiceManager

在整个远程服务调用过程中两个重要对象，一个是Binder对象，一个就是ServiceManager类，这个类是管理系统服务的类，他可以注册服务，查询服务，系统服务在系统启动的时候会通过addService进行服务注册，然后应用就可以通过getService进行服务查询，而在这个过程中，底层会维护一个这些服务的binder链表结构，同时每个服务的binder对象都一个句柄handle，通过这个句柄来表示通信标识，这样通信才不会紊乱。

4、底层通信核心Binder
底层真正实现跨进程通信的机制Binder，其实是通过虚拟驱动程序/dev/binder进行通信的。
