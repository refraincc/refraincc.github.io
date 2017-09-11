
---
title: Mac下的SVN分支管理
layout: post
date:  2017-01-02 11:39:55
author: "refraincc"
tags:
	- Coder
---

#Mac下的SVN分支管理
因为以前对项目的管理是用的git对代码进行版本控制,但现公司一直在用SVN，最近新项目提包上线，想换一种版本控制的方式，于是就想着用SVN的分支做一做
#1.创建目录
1.1	在SVN的根目录下创建如下文件夹目录
```bash
branchs 		//此目录主要是放一下分支
tag				//此目录可以放一些大的版本
trunk			//项目主分支
```
1.2	将这三个文件夹提交至远程仓库
#2.为`trunk`创建分支
在终端中输入
```bash
svn copy trunk/branchs/trunk_1.0
```
然后再提交至远程仓库
这样`trunk_1.0`的分支就创建好了
#3.在主分支与1.0分支中有差异的情况下合并文件
1.在`trunk_1.0`目录下创建一个`test.txt`文件，并提交至远程仓库
2.切换到trunk目录下，执行
```bash
svn merge ../branches/trunk_1.0/
```
3.这样就能看见在`trunk`目录下的`test.txt`文件了

