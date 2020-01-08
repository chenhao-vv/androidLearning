---
title: Git 
tags: git,小书匠
renderNumberedHeading: true
grammar_cjkRuby: true
---



## git 整体流程图
![git](./images/git.png)

## add命令
git add "文件名、文件夹名"
git add .  //添加全部

## commit命令
git commit -m  "提交message"
git  commit   //跳至默认编辑器，输入提交说明
git commit --amend   //将本次提交， 合并到上次提交之中（或者说修改本次提交）

## branch
git branch dev 	//新建dev分支
git branch -a  //查看所有分支，当前分支前有*显示
git branch   //查看本地分支
git branch -d dev  //删除dev分支
gitbranch -D dev  //强制删除dev分支，无论是否合入主线

## checkout 
git checkout  dev //切换到dev分支
git checkout commitid
git checkout HEAD^2

## reset 
git reset --soft  commitid  //重置到commitid提交，本地代码不变
git reset --hard commitid  //重置到commitid提交，本地代码也重置

## stash
git stash 
git stash save "message"

git stash apply "message"    //应用储存信息，但是不会清除
git stash pop "message"      //应用并清除

git stash list 
git stash clear

## status
git status 
git diff 
git diff "文件名"





参考：
[GitHub马蹄铁](https://github.com/veedrin/horseshoe?from=singlemessage)
[掘金博客: git基本操作，一篇文章就够了](https://juejin.im/post/5ae072906fb9a07a9e4ce596)
