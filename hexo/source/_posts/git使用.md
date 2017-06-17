title: git 笔记
date: 2015-10-01 14:54:34
tags: git
categories: note
---


## 安装及初始设置
```bash
$ git config --global user.name "Amy"
$ git config --global user.email "duyao_dy@163.com"
```
设置邮箱及姓名

## 创建仓库及添加文件
1. 切换到代码所在的目录下面
2. 创建仓库
```bash
git init
```
3. 添加文件到仓库
 3.1 `git add *`添加所有文件
     `git add test.txt`添加单个文件
 3.2 `git commit -m "first commit"`提交到`stage`区中

## 查看
`git statue`查看工作区状态

`git revert head`删除






