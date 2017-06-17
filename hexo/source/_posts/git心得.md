title: git心得
date: 2015-10-29 23:26:41
tags: git
categories: note
---
## 使用git的心得

<!--more-->

- 建库不增加contribution啊！！
  所以应该弄个不同文件夹每天更新
  contribution中每个小格子代表每一天

- blog更新也不算在git-Contributions！

- git 文件夹是灰色的，而且是空的
  因为文件中还是包含了`.git`，还是一个`repository`

- 本地建库，同步到远程

  1. 在本地准备好文件，确定文件家中不包含包含了`.git`
  2. 在远程建库，得到URL，本地`git init`
  3. `git remote add origin [https URL]`
  4. `git add --all .`添加所有
  5. `git commit -m "message"`提交到缓存
  6. `git push -u origin master`提交到远程，第一次添加需要加`-u`



