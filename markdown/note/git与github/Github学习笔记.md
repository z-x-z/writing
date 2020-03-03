# Github学习笔记

***

###1. Git本地仓库中文件的增删改操作

* 向仓库添加文件、目录:
  1. ``git add fileName | directorName``
  2. ``git commit -m "add ..."``
* 修改仓库中已有的文件、目录
  1. ``git add fileName | directoryName``（重新将目标添加入暂存区）
  2. ``git commit -m "modify ..."``
* 删除仓库已有的文件或目录
  1. ``git rm -rf fileName | directorName``（将目标从暂存区中删除）
  2. ``git commit -m "delete ..."``
  3. ``rm -rf fileName | directoryName``（可选，从工作区删除目标）

###2. Git本地仓库与远程仓库的连接

* 为本地仓库添加远程仓库
  1. 在Github创建一个名为repositoryName的仓库
  2. 利用命令``git remote add origin git@github.com:userName/repositoryName.git``添加远程仓库

* 将本地仓库的分支推送（push）至远程仓库origin中:

  * 推送master分支

    1. ``git push -u origin master``（-u使得**远程仓库(origin)的master分支**成为**本地仓库的当前分支**的上游，这使得以后在当前分支使用``git pull``会直接从origin的master分支中获取内容。）

    2. 输入ssh秘钥(如果设有)
    
  * 推送其他分支

    1. ``git push -u origin 分支名``
    2. 输入ssh秘钥(如果设有)