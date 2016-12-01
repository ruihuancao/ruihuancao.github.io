---
title: Flask使用
date: 2014-10-3 20:10:20
categories:
- 开发
- Flask
tags:
- Python
- Flask
---

Flask是python的一个web框架，简洁，灵活，插件可配置
<!--more-->

## Flask
 flask文档地址：http://docs.jinkan.org/docs/flask/

## 安装
pip 安装 http://pip-cn.readthedocs.io/en/latest/installing.html
```
python get-pip.py
```

## virtualenv 安装
```
sudo pip install virtualenv
#或者
sudo apt-get install python-virtualenv
```

创建虚拟环境

```
virtualenv -p /usr/bin/python2.7 ENV2.7# 创建虚拟环境 指定python版本为2.7
source ./bin/activate # 激活虚拟环境
deactivate # 退出虚拟环境
pip install -r requirement.txt # 安装依赖
```
## 全局安装
```
sudo pip install Flask
```

## Flask-RESTful
  Flask-RESTful 文档地址：http://www.pythondoc.com/Flask-RESTful/quickstart.html

  http://[hostname]/[name]/api/v1.0/
