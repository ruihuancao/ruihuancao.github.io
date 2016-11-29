---
title: Android源码编译
date: 2016-08-18 12:08:59
categories:
- 开发
- Android
tags:
- Android
---

------

之前编译过源码，刷了几台机器，还错机器导致数据被清除了，只好重新下载了，正好重新记录下。

主要有以下几个目的

> * 编译源码，制作rom刷机
> * 引入Android studio 查看源码
> * 学习源码

# 编译源码，制作rom刷机

[AOSP文档](https://source.android.com/source/initializing.html)

[清华镜像文档](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)

编译机器：Ubuntu14.0.4

刷入机器：LG的nexus5x
## 建立环境

### 1.安装openjdk 配置环境

```
# 下载openjdk
wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openjdk-8/openjdk-8-jre-headless_8u45-b14-1_amd64.deb
#  SHA256 0f5aba8db39088283b51e00054813063173a4d8809f70033976f83e214ab56c0
wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openjdk-8/openjdk-8-jre_8u45-b14-1_amd64.deb
# SHA256 9ef76c4562d39432b69baf6c18f199707c5c56a5b4566847df908b7d74e15849
wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openjdk-8/openjdk-8-jdk_8u45-b14-1_amd64.deb
# SHA256 6e47215cf6205aa829e6a0a64985075bd29d1f428a4006a80c9db371c2fc3c4c

# 校验文件完整性
sha256sum {downloaded.deb file}

# 安装openjdk
sudo apt-get update
sudo dpkg -i openjdk-8-jre-headless_8u45-b14-1_amd64.deb
sudo dpkg -i openjdk-8-jre_8u45-b14-1_amd64.deb
sudo dpkg -i openjdk-8-jdk_8u45-b14-1_amd64.deb

# 安装中出错处理,安装依赖,再次安装即可
sudo apt-get -f install

# 更新环境配置
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

### 2.安装必需的依赖包
```
sudo apt-get install git-core gnupg flex bison gperf build-essential \
 zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
 libgl1-mesa-dev libxml2-utils xsltproc unzip
```
### 3.配置Usb可访问
```
$ wget -S -O - http://source.android.com/source/51-android.rules | sed "s/<username>/$USER/" | sudo tee >/dev/null /etc/udev/rules.d/51-android.rules; sudo udevadm control --reload-rules
```

## 下载源码

### 1.下载repo并配置
```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
### 2.初始化
有两中中方式可以初始化

#### 使用初始化包初始化
```
wget https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
tar xf aosp-latest.tar
cd AOSP   # 解压得到的 AOSP 工程目录
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
repo sync # 正常同步一遍即可得到完整目录
# 或 repo sync -l 仅checkout代码
```
#### 传统初始化方式
- 新建工作目录
```
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
```
- git 配置
```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
- 初始化
```
#清华镜像
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
#google 官方
repo init -u https://android.googlesource.com/platform/manifest
repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
```
- 同步
```
repo sync
```

## 编译源码
### 1.清理
```
make clobber
```
### 2.设置环境
```
source build/envsetup.sh
```
### 3.选择目标
// 这里需要根据自己的机器选择对应的版本，并查看对应的驱动文件下载到源码目录，直接运行，会生成vendor目录
```
lunch aosp_arm-eng
```
### 4.编译
```
make -j4
```
### 5.运行模拟器
```
emulator
```
### 6. 刷入真机中
刷如
