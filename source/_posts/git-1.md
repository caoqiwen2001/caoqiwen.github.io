---
title: git操作指令
date: 2018-07-06 15:34:19
tags:
---
###  git操作指令
git用于代码管理，多版本多分支管理操作，在多人管理的过程中还是用的比较多的。以下记录个人用的比较多的git命令。

 增加操作

```
git add .
```
如果只是修改了文件，并没有增加到暂存区，用这个指令进行撤回操作：
```
 git checkout -- 1.txt
```
如果有进行add操作到缓存区，那么可以用reset指令将暂存区的代码进行复原。

```
git reset --head
```
如果进行了commit操作，需要进行回退的话也是这个命令：

```
git reset -- hard Head
```
Head就是上一个版本，要回退到具体的版本可以执行

```
git log  【查看commit的提交记录】 
```

查看对应版本的哈希值，取前面几位就可以了。

回退到某一个版本
```
git reflog   【查看本地影响head指针命令记录,这个不会同步到远程仓库】
git reset --hard 版本号
```
选择远程仓库中的代码到本地分支

```
git checkout -b dev origin/dev在本地创建分支dev并切换到该分支，分支命名为dev，
```
拉取远端的代码到当前分支

```
git pull origin dev
```
本地切换分支到master

```
git checkout  master

```
<font color="green"> 合并master分支到当前分支上 </font>

```
git merge  master
```
 <font color="red">如果遇到有冲突，则需要解决冲突之后在执行以下两个命令</font>

```
git add 
git commit 
```
之后可以进行再次合并，然后在push到远程分支上。





