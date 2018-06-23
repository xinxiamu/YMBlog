---
title: git打标签封存版本
date: 2018-06-23 17:40:01
categories: git
tags: git-tag
---

在开发项目中，当我们的项目测试通过，并准备发布到生产环境时候，那么这是一个稳定的版本，此时，我们就需要封存该版本代码。做一个版本管理。本文就是介绍git下如何封存版本……

## 创建标签

1. 创建本地标签

    `git tag -a V1.0 -m '版本1.0，基于spring-boot 1.x'`
    
2. 查看标签

    `git tag`
    
    只看到版本信息`V1.0`，没看到备注信息。可以执行下面命令查看更加详细信息：
    
        mutian@mutian-ThinkPad-T440p:~/dev/java/github/ymu-framework$ git show V1.0
        tag V1.0
        Tagger: xinxiamu <932852117@qq.com>
        Date:   Sat Jun 23 17:45:17 2018 +0800
        
        版本1.0，基于spring-boot 1.x 
        
            
## 提交标签到服务器

上面创建的`tag`只是本地`git`仓库。下面把它推送到远程服务器。

    mutian@mutian-ThinkPad-T440p:~/dev/java/github/ymu-framework$ git push origin --tags 
    Username for 'https://github.com': xinxiamu
    Password for 'https://xinxiamu@github.com': 
    Counting objects: 1, done.
    Writing objects: 100% (1/1), 192 bytes | 0 bytes/s, done.
    Total 1 (delta 0), reused 0 (delta 0)
    To https://github.com/xinxiamu/ymu-framework.git
     * [new tag]         V1.0 -> V1.0
    
这个时候，突然发现一个重大bug，需要重新打版本，可以按下面方法来出来：

- 删除本地标签： `git tag -d V1.0`    

- 推送同名的空的标签到远程服务器，达到删除的目的：
    
    `git push origin :refs/tags/V1.0`
    

    mutian@mutian-ThinkPad-T440p:~/dev/java/github/ymu-framework$ git push origin :refs/tags/V1.0
    Username for 'https://github.com': xinxiamu
    Password for 'https://xinxiamu@github.com': 
    To https://github.com/xinxiamu/ymu-framework.git
     - [deleted]         V1.0

- 拉取并切换到对应分支，在该分支上修复bug。
    
    如果远程服务器没有该分支，则可以基于该`tag`创建分支。并在新建分支上修改bug。
    
    `git checkout -B test v0.1.0` 强制创建一个基于指定的tag的分支。
    
    `test`为分支名称，`v0.1.0`为标签。

- 按照上面步骤重新打标签并推送到远程服务器封存。
- 合并该分支修改到主干分支`master`上，否则下次该bug还是存在。

## 获取远程指定的封存版本

更适合运维，拉取指定稳定版本进行发布到生产环境。

`git fetch origin tag V1.0`

## 切换到tag

`git checkout V1.0`

----------------------------------------------------------------

## git 获取指定的tag处代码（来源网络）

tag是对历史提交的一个id的引用，如果理解这句话就明白了tag的含义

使用git checkout tag即可切换到指定tag，例如：git checkout v0.1.0

切换到tag历史记录 会使当前指针处在分离头指针状态，这个时候的修改是很危险的，在切换回主线时如果没有合并，之前的修改提交基本都会丢失，如果需要修改可以尝试git checkout -b branch tag创建一个基于指定tag的分支，例如：git checkout -b test v0.1.0 这个时候就在这个test分支上进行开发，之后可以切换到主线合并。

注意这时候的test分支的代码很多都是tag版本处的，但是test分支head节点在最前面，这时候切换到主线进行合并，要注意合并后的代码冲突问题，不要让旧代码覆盖了主线的新代码。

git checkout -B
这个命令，可以强制创建新的分支，为什么加-B呢？如果当前仓库中，已经存在一个跟你新建分支同名的分支，那么使用普通的git checkout -b 这个命令，是会报错的，且同名分支无法创建。如果使用-B参数，那么就可以强制创建新的分支，并会覆盖掉原来的分支。

git checkout -B test v0.1.0 强制创建一个基于指定的tag的分支。


