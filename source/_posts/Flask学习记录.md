---
title: Flask学习记录
date: 2016-10-3 20:10:20
categories:
- Flask
tags:
- Python
- Flask
---

Flask 是一个 Python 实现的 Web 开发微框架。

flask文档地址：http://docs.jinkan.org/docs/flask/
<!--more-->

## 安装
### pip
［pip地址](http://pip-cn.readthedocs.io/en/latest/installing.html)
```
python get-pip.py
```
### virtualenv
virtualenv 为每个不同项目提供一份 Python 安装。它并没有真正安装多个 Python 副本，但是它确实提供了一种巧妙的方式来让各项目环境保持独立。
```
sudo pip install virtualenv
#sudo apt-get install python-virtualenv
# 创建虚拟环境 指定python版本为2.7
virtualenv -p /usr/bin/python2.7 python2.7
# 创建虚拟环境 指定python版本为3.5
virtualenv -p /usr/bin/python3.5 python3.5
#激活虚拟环境
source ./python3.5/bin/activate
# 退出虚拟环境
deactivate
# 安装依赖
pip install -r requirement.txt

#生成requirements.txt文件
pip freeze >requirements.txt
```
### Flask
```
sudo pip install Flask
```
## Flask扩展
Flask 有很多可用扩展
### flask_script
命令管理
```
pip install flask_script
```
### flask_bootstrap
bootstrap扩展
```
pip install flask_bootstrap
```
### flask_moment
时间格式化
```
pip install flask_moment
```
### flask_wtf
flask 表单扩展
```
pip install flask_wtf
```
### flask_sqlalchemy
数据库管理
```
pip install flask_sqlalchemy
```
### flask_migrate
数据库升级
```
pip install flask_migrate
```
### Flask-Login
```
pip install Flask-Login
```
