---
title: docker学习-第六课：三剑客之Docker Compose
date: 2018-08-01 20:33:32
categories: docker
tags: docker-compose
---

在一个例如web应用中，除了应用本身之外，其还关联用到数据库等……，如果都采用docker化的话，那么就需要启动所有相关的docker容器。那么，有没有可能，按照某种顺序，对所有相关联的docker容器进行编排，然后按照一定的先后顺序依次启动。是的，Docker Compose就是这么一个工具，容器编排工具。定义和运行多个容器的应用。 

Compose中有两个重要的概念：

- 服务（Service）：一个应用的容器,实际上可以包括若干运行相同镜像的容器实例。
- 项目(project)：由一组关联的应用容器组成的一个完整业务单元,在	docker-compose.yml文件中定义。

Compose	的默认管理对象是项目,通过子命令对项目中的一组容器进行便捷地生命周期管理。

    