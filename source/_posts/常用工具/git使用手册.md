---
title: git使用手册
tags: 常用工具
categories: 常用工具
cover: https://gitee.com/studentgitee/note-picture/raw/master/xd16.png
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

## 1、git工作原理图



![img](https://www.runoob.com/wp-content/uploads/2015/02/git-command.jpg)

**说明：**

- workspace：工作区
- staging area：暂存区/缓存区
- local repository：版本库或本地仓库
- remote repository：远程仓库



## 2、git常用命令

Git 常用的是以下 6 个命令：**git clone**、**git push**、**git add** 、**git commit**、**git checkout**、**git pull**。

```git
git clone 仓库地址  #拉取master分支的代码,拷贝一份远程仓库，就是下载远程仓库的一个项目。

git init newtest #使用指定的目录作为git仓库,初始化后，会在 newtest 目录下会出现一个名为 .git 的目录，所有 Git 需要的数据和资源都存放在这个目录中

git remote add origin https://gitee.com/studentgitee/notes-center.git #设置推送到的远程仓库地址

git pull --rebase origin master #拉取远程仓库中master分支的代码，下载远程代码并合并

git add *.java   #提交代码到暂存区

git commit -m '提交代码说明，比如用户管理功能提交' #提交代码暂存区到本地仓库

git push -u origin master #提交代码到远程仓库的master，上传远程代码并合并
```



## 3、设置提交代码时的用户信息

```
git config --global user.name "xxx"
git config --global user.email xxx@qq.com
```

