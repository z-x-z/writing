# Docker技术学习笔记

* ##介绍

  1. 作用:

     Docker，可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

* ## Docker框架

  Docker 包括三个基本概念:

  - **镜像（Image）**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
  - **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的<u>类和实例</u>一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
  - **仓库（Repository）**：仓库可看着一个代码控制中心，用来保存镜像。
  
* system级别操作

  * 查看操作

    ``docker system df``: 查看Dokcer空间占用情况

  * 删除操作:

    ``docker system prune``: 删除以下实体

    * 已停止的容器
    * 未被任何容器使用的卷
    * 未被任何容器所关联的网络
    * 所有悬空的镜像

    ``docker system prune -a``: 清理所有未被使用的实体(慎用！！)

* 镜像

  * 列出镜像: dcoker images

  * 删除镜像: 

    * docker rmi 镜像名:标签名

    * docker rmi $(docker images -q -f dangling=true):

      清理所有悬空镜像（镜像没有名称与标签）

  * 更新镜像: 在又某镜像创建的容器中，我们可以利用该容器来更新其基础镜像。
    * 在运行的容器中使用``apt-get update``命令进行更新
    * 输入``exit``退出该容器
    * 在终主机端中使用``docker commit -m="描述信息..." -a="作者信息..." 容器ID 指定要创建的容器名:标签名``
    
  * 构建镜像: docker build -t 镜像名:标签名 . 
    * -t :指定目标镜像名称
    * . :指定Dockerfile为当前目录

* 容器操作

  * 创建容器

    ```shell
    docker run -it nginx:latest /bin/bash
    ```

    * docker run [option] image:label [command] [args]
    * [option]:
      * -it: 以交互式方式创建容器并为其分配一伪输入终端
      * -p: 端口映射，格式为``主机端口:容器端口``
      * --name="containerName": 指定容器名称
      * 

  * 清理容器

    * docker rm + 容器名: 清理容器
    * docker rm $(docker ps -a -q): 清理所有容器

  * 进入容器:

    * docker attach 容器ID
    * docker exec 容器ID

    

    