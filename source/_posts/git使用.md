---
title: git使用
date: 2016-08-16 11:40:03
categories: 开发
tags: Test
---

git 常用命令记录

## 下载安装
  参考官网[git地址](https://git-scm.com/downloads/)

<!--more-->

## 配置

```
git config --global user.name "ruihuancao"
git config ==global user.email "ruihuancao@gmail.com"
git config --global core.editor vim
```

## 管理多个ssh key 配置

对于github和公司gitlab 使用不同的key

```
# 生成公司gitlab的ssh key,选择保存目录到~/.ssh/id_gitlab_rsa
ssh-keygen -t rsa -C "gongsi@mail.com"
# 生成个人github的ssh key,选择保存目录到~/.ssh/id_github_rsa
ssh-keygen -t rsa -C "ruihuancao@gmail.com"
#新建config文件
touch  ~/.ssh/config
#写入如下配置
# gitlab
Host gitlab.gongsi.net
    HostName gitlab.gongsi.net
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_gitlab_rsa
    User caoruihuan
# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_github_rsa
    User caoruihuan
#测试
ssh -T git@github.com
```

## 常用命令

```
创建仓库： git init
添加文件到仓库： git add <file>
提交到仓库： git commit -m "说明"
查看状态： git status
查看区别： git diff
查看日志： git log
HEAD指向的版本就是当前版本,版本切换： git reset --hard commit_id
重返未来：git reflog 查看历史命令，确定切换到哪个提交
撤销文件修改: git checkout -- file
返回修改：git reset HEAD file
删除文件：git rm file
关联远程库：git remote add origin git@server-name:path/repo-name.git；
推送分支： git push -u origin master （第一次推送master分支的所有内容）
推送到master主分支：git push origin master
克隆仓库： git clone
查看分支： git branch
创建分支： git branch <branch name>
切换分支： git checkout <branch name>
创建+切换分支： git checkout -b <branch name>
合并某分支到当前分支： git merge <branch name>
删除分支： git branch -d <branch name>   
强行删除：git branch -D <branch name>
保存现场： git stash
恢复现场并删除记录： git stash pop
查看所有标签：git tag
新建标签(默认当前Head)： git tag <tag name>
指定commit id 新建tag； git tag <tag name> commit_id
git tag -a <tagname> -m "描述信息"
推送标签：git push origin <tagname>
推送本地所有为推送标签：git push origin --tags
删除本地标签：git tag -d <tagname>
删除远程标签（先删除本地，然后推送）：git push origin :refs/tags/<tagname>
修改颜色：git config --global color.ui true
git
添加.gitignore 文件，该文件需要添加到版本库
```
