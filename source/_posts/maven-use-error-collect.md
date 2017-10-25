---
title: maven使用常见错误收集
date: 2017-10-25 14:41:24
categories: maven
tags: maven-error
---

本文收集使用maven过程中的常见错误。

> - 问题：
maven继承，使用 *`<relativePath></relativePath>`*，出现找不到指向pom，但是实际上已经正确指向了。 
- 解决：
    The relative path of the parent pom.xml file within the check out. If not specified, it defaults to ../pom.xml. Maven looks for the parent POM first in this location on the filesystem, then the local repository, and lastly in the remote repo. relativePath allows you to select a different location, for example when your structure is flat, or deeper without an intermediate parent POM. However, `the group ID, artifact ID and version are still required,` and must match the file in the location given or it will revert to the repository for the POM. This feature is only for enhancing the development in a local checkout of that project. Set the value to an empty string in case you want to disable the feature and always resolve the parent POM from the repositories.
  Default value is: ../pom.xml.
  *所以，要在每个父pom上都要加上groupId,artifactId,version。搞定。*
    
    
