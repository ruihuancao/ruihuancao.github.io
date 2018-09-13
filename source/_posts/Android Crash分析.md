---
title: Android Crash分析
date: 2017-01-18 12:08:59
categories:
- Android
tags:
- Android
---
Android开发中，程序Crash分三种情况：
- 未捕获的异常
- ANR（Application Not Responding）
- 闪退（NDK 程序引发错误）

<!--more-->

# 未捕获的异常
其中未捕获的异常根据 logcat 打印的堆栈信息很容易定位错误。

# ANR（Application Not Responding）
ANR(Application Not responding)，是指应用程序未响应，Android系统对于一些事件需要在一定的时间范围内完成，如果超过预定时间能未能得到有效响应或者响应时间过长，都会造成ANR,并会在/data/anr目录下生成一个traces.txt文件，记录系统产生anr异常的堆栈和线程信息。一般地，这时往往会弹出一个提示框，告知用户当前xxx未响应，用户可选择继续等待或者Force Close。

## ANR类型
- KeyDispatchTimeout(5 seconds) –主要类型按键或触摸事件在特定时间内无响应
- BroadcastTimeout(10 seconds) –BroadcastReceiver在特定时间内无法处理完成
- ServiceTimeout(20 seconds) –小概率类型 Service在特定的时间内无法处理完成

## ANR产生的原因
应用程序自身引起的。如主线程阻塞（block）、挂起、死锁、死循环、IO长时间读写，执行耗时操作等

其他应用进程引起的。其他进程的CPU占用率过高、某一时刻系统CPU负载过高等，都会导致当前进程无法抢到CPU时间片。

## ANR定位
系统发生ANR的时候，一般会以以下三种方式记录
- logcat
- /data/anr/traces.txt
- dropBox

主要查看anr类型，cpu使用情况，发生位置，具体分析需要跟进具体业务情况

## 避免ANR
1. 编写代码时注意
2. StrictMode
3. monkey

# 闪退（NDK 程序引发错误）

# Tombstone
## Linux 信号机制
信号机制是 Linux 进程间通信的一种重要方式，Linux 信号一方面用于正常的进程间通信和同步.另一方面，它还负责监控系统异常及中断。 当应用程序运行异常时， Linux 内核将产生错误信号并通知当前进程。当 Linux 应用程序在执行时发生严重错误，一般会导致程序 crash。其中，Linux 专门提供了一类 crash 信号，在程序接收到此类信号时，缺省操作是将 crash 的现场信息记录到 core 文件，然后终止进程。

## 什么 Tombstone？
Android Native 程序本质上就是一个 Linux 程序，因此当它在执行时发生严重错误，也会导致程序 crash，然后产生一个记录 crash 的现场信息的文件，而这个文件在 Android 系统中就是 tombstone 文件。

tombstone 文件位于路径 /data/tombstones/ 下，
tombstone 文件它主要由下面几部分组成：

- Build fingerprint
- Crashed process and PIDs
- Terminated signal and fault address
- CPU registers
- Call stack
- Stack content of each call

## 解析Tombstone文件
- addr2line
- ndk-stack
