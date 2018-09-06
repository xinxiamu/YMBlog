---
title: git使用
date: 2018-09-06 18:08:06
categories: git
tags: git问题
---

本文介绍git的使用方法以及一些问题……

## 回滚版本库
在`git commit`多次后，版本库又会记录，但是在某次提交当中，提交了错误的代码，或者过大的文件，这个时候，就要做撤销，回滚。

方式一：    
`git log`   #拿到版本库id    
`git reset 版本库id`
