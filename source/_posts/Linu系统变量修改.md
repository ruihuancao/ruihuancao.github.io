---
title: Android 反编译纪录
date: 2015-04-28 20:20:20
categories:
- 开发
- linux
tags:
- linux
---

linux配置系统环境变量有3中方式

## 修改/etc/profile文件
所有用户有效
```
# 添加新的变量
vim /etc/profile
# 生效
source /etc/pprofile
```
## 修改.bashrc文件
用户级别有效
```
vim ~/.bashrc
source ~/.bashrc
```
## 直接在shell下设置变量
在单个终端窗口有效
```
export JAVA_HOME=/usr/share/jdk1.8
```
