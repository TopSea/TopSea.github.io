---
layout: post
title: Git 常用命令
date: 2021-6-28
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: git.jpg # Add image post (optional)
tags: [Android, 游戏, Java] # add tag
---
#Git 常用命令
虽然现在大部分git操作都图形化了，但是有些git命令没有实现，而且大部分都是救急的。   
`git init`: 把初始化为Git仓库。   
`git init [projectName]`: 新建一个projectName目录并初始化为Git仓库。   
`git clone [address]`: 克隆远程Git仓库。   

`git config --list`: 列出当前所有的Git配置。

`git branch [newBranchName]`: 新建分支。   
`git branch -D [branchName]`: 删除分支。   
`git add [fileName]`: 添加文件到暂存区。   
`git rm [fileName]`: 删除文件。   
`git pull [remoteHostName] [remoteBranchName] [localBranchName]`: 把远程仓库的指定分支pull到本地，再与本地的指定分支合并。   
`git diff [branchName]`: 当前分支与指定分支的比较。   
