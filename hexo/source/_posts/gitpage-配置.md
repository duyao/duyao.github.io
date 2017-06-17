title: gitpage 配置
toc: true
date: 2016-12-19 14:37:29
tags:
categories: note
---
## git中的两个分支

设置完成gitpage后，要再新建一个分支，用来存放hexo下面的配置及文件，这样电脑重装系统后，只需要下载相关组件，而不用重新配置
即在`duyao.github.io`下面有两个分支，一个是`master`，另一个是`hexo`
我在hexo中把blog内容更新到`master`中
而在`hexo`分支中存放的是hexo的配置文件和`master`的内容
因此对于日常的写博客，首先使用 `hexo g -d`发布，这时候内容是同步到`master`中
然后用`git push origin hexo`将这些内容同步到`hexo`中

## git快捷键

有两种方法设置快捷键，一个是通过`.gitconfig`,另一种是`.bashrc`。

- 在`.gitconfig`添加如下内容
```
[user]
	name = Amy
	email = duyao.dy1993@gmail.com

...

[alias]
    aa = add .
    cm = commit -m
    c = commit
    co = checkout
    dt = difftool
    mt = mergetool
    po = push origin
    pom = push origin master
    praise = blame
    ff = merge --ff-only
    st = status
    sync = !git pull && git push
```

- 在c://User/user中建立文件`.bashrc`，并添加如下内容
```
alias gs='git status'
alias gd='git diff'
alias ga='git add'
alias gc='git commit'
alias gck='git checkout'
alias gb='git branch'
alias gl='git log'
alias gthis='git rev-parse --abbrev-ref HEAD'
alias gpushthis='git push origin `gthis`'
alias gpullthis='git pull origin `gthis`'
alias gup='git remote update'
alias gpl='git pull origin'
```
