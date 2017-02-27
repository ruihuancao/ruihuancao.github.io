---
title: Android 系统启动
date: 2017-01-18 12:08:59
categories:
- 开发
- Android
tags:
- Builder
---

1 init 进程
2 init启动Zygote进程
3 由此进入java代码层，加载系统运行依赖的class类和依赖的资源文件，然后启动系统的
	核心服务SystemServer
4 SystemServer加载本地的动态链接库，然后调用动态链接库中的c函数，这时代码执行到了Libraries层
5 启动硬件相关的服务、获取Android运行时调用SystemServer下的init2
6 Android系统服务的初始化， 并且把服务添加ServiceManager中, 便于以后系统进行统一管理.  
7 ActivityManagerService启动第一个界面
