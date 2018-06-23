---
title: git分支管理
date: 2018-06-23 15:29:33
categories: git
tags: git分支、github分支
---

在项目开发过程中，为了更好的开发，管理上需要对代码做版本管理。一般情况下，在开发之前就理应设定开发版本，然后开发人员在对应版本上编写代码，管理者再在某个时刻把分支`branch`代码合并回主分支。在最终稳定一个版本后，再打`tag`做版本永久保存。

首先，要当前目录为`git`仓库。`git`仓库拉取下来默认是`master`分支。

## 创建本地分支

1. 查看有哪些分支

    `git branch`

2. 创建一个分支

    `git branch name1` `name1`为分支名称。不能和`tag`的相同
    
3. 切换到分支

    `git checkout name1`
    
实际上，2和3步骤也可以一步到位：`git checkout -b name`。创建并切换到分支

下面，就可以在该分支上进行文件操作了。

_注意_:如果用 git checkout master切换到主分支，在当name分支下进行的文件变更的内容无法看到。当切回name分支后，又可以看到了。

## 提交分支到服务器

`git push origin name1`

说明：提交分支到服务器后，在本地分支进行文件变更后，可以执行上面同样命令，将变更信息更新到服务器上的该分支。

## 将分支变更的内容合并到master分支

1. 切换到master分支： `git checkout master`

2. 合并`name1`分支到当前`master`分支： `git merge name1`

_注意_: 这个时候合并到`master`分支上的内容还没提交到服务器的，需要`push`提交。

## 删除分支

1. 删除本地分支：`git branch -d name1`
2. 删除服务器上的分支： `git push origin :name   (分支名前的冒号代表删除)`  

## clone分支

`git`仓库拉取下来，默认会把所有内容`clone`下来。 
但是默认只创建`master`分支，需要执行`git branch -r`才能看到所有分支名字。  
想把其它分支拉取下来，执行： `git checkout 分支名`。这样就把远程的分支拉取下来了。  
再执行`git branch`，就能看到本地所有的分支了。然后可以切换分支编辑。