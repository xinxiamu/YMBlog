---
title: 如何直接在github网站上更新你fork的repo
date: 2017-10-13 09:39:58
categories: github
tags:
---
玩过github的人一定会在你自己的账号上fork了一些github开源项目。这些开源项目往往更新比较活跃，你今天fork用到你自己的项目中去了，过几个星期这个fork的origin可能有一些bugfix了，你怎么办呢？当然直接到Origin repo中去clone是一个方法，但是github的public repo有可能过一段时间就被作者删除了，你是否希望在origin即使已经被删除的情况下，你的账号下依然有你钟情的repo？

解决上面的问题，最好的方法就是不定时地将origin的commit sync到你自己的fork repo中，一方面能够保持鲜活，另一方面有备无患。那么如何sync呢？又有几种方案，一种是你直接在本地clone的repo中，pull upstrame,做好merge，随后push到你自己的fork repo中。另外还有一种更加简便聪明的方法：只需在github网站上点几个鼠标，不用本地开发环境轻松搞定：

> 1.打开你的github fork repo;
 2.点击Pull request;
3.点击new pull request.默认情况下，github会比较original/your fork，这时应该不会有任何输出，因为你并没有做过任何变更；
4.点击switching the base.这时github将反过来比较yourfork/original，这时你将看到original相对你fork时的所有commit;
5.点击create a pull request for this comparison，这时将会反过来向你的repo提交一个pull request;
6.这时你作为你自己fork的repo的owner，你就可以点击confirm the merge，大笔一挥，所有的改动都被你一网打尽了@！
enjoy it!

附上比较费劲的另外一种更新办法：

    git remote add upstream <pathtooriginalrepo>
    
    git fetch upstream
    
    git merge upstream/master master
    
    git push origin master
