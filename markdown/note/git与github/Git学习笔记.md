# 手撕Git，告别盲目记忆

***



文章出自：https://zhuanlan.zhihu.com/p/98679880

## 文章导读

- Git的分区（工作区，暂存区，版本库）
- Git的原理
- Git分支
- 版本的回滚（revert，reset）
- 代码暂存（stash）

## 小概述-何为Git

> Git是一个分布式版本控制系统，为了快速高效地处理小到大型项目的所有内容。通过对信息的压缩和摘要，使得所占空间非常小，但能够支持项目版本迅速迭代的开发工具。

### **一、Git的分区**

本章主要从基础入手，先介绍git的分区。

**1.1 三大分区**

- 工作区，也叫工作树，Working Directory
- 暂存区，也叫stage，index
- 版本库，也叫本地仓库，commit History

当我们把代码从git hub档下来或者说初始化git项目后，便有了这三个分区的概念。

文件在Git不同分区中的表现：

![img](https://pic1.zhimg.com/80/v2-8288c8c09056ce308f20f781f447fd6c_720w.jpg)

**工作区**
工作区应该不陌生，就是我们能看见，直接编辑的区域。对于一些新增的文件，如果没有被add到暂存区，就会以红色的形式放置在工作区。

**暂存区**
数据暂时存放的区域，对于**add** git版本控制的文件，就算是进入暂存区啦。可以理解为数据进入本地代码仓库之前存放的区域。由于还没对本地仓库生效，所以是数据暂时存放的区域。

对暂存区的文件修改后，会以蓝色的形式显示。如果第一次创建并add到暂存区的文件，由于远程仓库没有同步，所以会显示绿色。

注：存放在 ".git目录下" 下的index文件（.git/index）中

**版本库**
在暂存区**commit**的代码会被放入版本库中。可以理解为一个本地的代码仓库，push的时候，才会把版本库的数据全都发送到远程仓库中。

注：存放在工作区中“.git”目录下。

![img](https://pic1.zhimg.com/80/v2-1679362b21db8776fc27a7b1d042dca4_720w.jpg)图片来源于网络

扩展阅读：
[https://juejin.im/post/5b6c4eeff265da0f4d0da3fa](https://link.zhihu.com/?target=https%3A//juejin.im/post/5b6c4eeff265da0f4d0da3fa)
[https://www.runoob.com/git/git-workspace-index-repo.html](https://link.zhihu.com/?target=https%3A//www.runoob.com/git/git-workspace-index-repo.html)

### **1.2** 涉及指令

**1.2.1 分区转换指令**

**git add**
数据从工作区转移至暂存区

**git commit**
数据从暂存区转移至版本库，也就是本地仓库

**git push**
数据从版本库中发送到远程仓库

指令太多？一张图就能记下～

![img](https://pic1.zhimg.com/80/v2-8e73605a803bd29ceacd86923e08f5f0_720w.jpg)

**1.2.2 分区对比指令**

**git diff**
工作区与暂存区对比

**git diff head**
工作区与版本库对比

**git diff --cached**
暂存区与版本库对比

指令太多？一张图就能记下～

![img](https://pic2.zhimg.com/80/v2-e95ac3baeb28734107f8bec468525661_720w.jpg)

## 二、Git的原理

操作Git代码库前，一定要了解Git是怎么记录每次提交的代码变化的？换句话说，每一次commit在保证开发效率的前提下，都提交了什么？

### **2.1 git如何存储文件/目录信息**

首先我们使用`git init`，初始化一个新的git项目。这个目录会在项目的根目录下创建.git的隐藏目录，相信大家都不陌生。

```powershell
MacBook-Pro:wuya eleme$ git init
已初始化空的 Git 仓库于 /Users/eleme/wuya/.git/
```

然后查看一下.git的目录树

```powershell
MacBook-Pro:wuya eleme$ tree -a
.
└── .git
    ├── HEAD
    ├── config
    ├── description
    ├── hooks
    │   ├── applypatch-msg.sample
    │   ├── commit-msg.sample
    │   ├── fsmonitor-watchman.sample
    │   ├── post-update.sample
    │   ├── pre-applypatch.sample
    │   ├── pre-commit.sample
    │   ├── pre-push.sample
    │   ├── pre-rebase.sample
    │   ├── pre-receive.sample
    │   ├── prepare-commit-msg.sample
    │   └── update.sample
    ├── info
    │   └── exclude
    ├── objects
    │   ├── info
    │   └── pack
    └── refs
        ├── heads
        └── tags

9 directories, 15 files
```

我们会发现，有一个叫Objects的目录。这个目录就是**存储文件变化的核心**。我们往工作区中存入一个测试文件a.md和一个test文件夹并查看objects发生的变化。

```powershell
MacBook-Pro:wuya eleme$ echo 'test1' > a.md
MacBook-Pro:wuya eleme$ mkdir test
MacBook-Pro:wuya eleme$ echo 'test2' > test/b.md
MacBook-Pro:wuya eleme$ git add a.md test
MacBook-Pro:wuya eleme$ tree -a .git/objects
.git/objects
├── 18
│   └── 0cf8328022becee9aaa2577a8f84ea2b9f3827
├── 9d
│   └── aeafb9864cf43055ae93beb0afd6c7d144bfa4
├── info
└── pack

4 directories, 2 files
```

注意，文件夹放入到暂存区后，并不会马上在objects中显示，commit后才会。此时多了两个文件，其实就是修改过的两个文件以及修改内容。

Objects下存放的**文件名**就是根据SHA1算法哈希的“指纹”，为了能够在本仓库中和其他文件区分出来。**文件内容**就是Git将信息压缩后形成的二进制文件。
通过`git cat-file [-t] [-p]`，可以看到Object的类型与文件的内容。

```powershell
MacBook-Pro:wuya eleme$ git cat-file -t 9dae
blob
MacBook-Pro:wuya eleme$ git cat-file -p 9dae
test1
```

通过`git hash-object a.md`能够显示该文件在本仓库生成的hash值，与之前的目录树显示是对应的。

```powershell
MacBook-Pro:wuya eleme$ git hash-object a.md
9daeafb9864cf43055ae93beb0afd6c7d144bfa4
```

### **2.2 git Object的类型**

git Object有三种类型：

- Blob
- Tree
- Commit

简单来说，**文件**都被存储为Blob类型，**文件夹**则为Tree类型，每次**提交的节点**被存储为Commit类型数据。因此，Git会以这三种类型来存储我们的文件。简单看下目录存储的映射关系：

![img](https://pic1.zhimg.com/80/v2-b1da1f9d9ba4212b80a98000e80f75d4_720w.jpg)

初步猜想，如果把这些文件都commit到代码库，objects目录应该会有4个目录。即2个blob，1个tree，1个commit。

```powershell
MacBook-Pro:wuya eleme$ git commit -a -m "加入到代码库中，观察objects目录变化"
[master（根提交） a16b538] 加入到代码库中，观察objects目录变化
 2 files changed, 2 insertions(+)
 create mode 100644 a.md
 create mode 100644 test/b.md
MacBook-Pro:wuya eleme$ tree -a .git/objects
.git/objects
├── 18
│   └── 0cf8328022becee9aaa2577a8f84ea2b9f3827
├── 21
│   └── d0758079bdf2c8f7514687174454c804eb0c74
├── 9d
│   └── aeafb9864cf43055ae93beb0afd6c7d144bfa4
├── a1
│   └── 6b5382a9b646a7df8d21301391f29b2f7bfb65
├── a7
│   └── 6c93bb75184ef4b34c88a301c2351ae2219407
├── info
└── pack

7 directories, 5 files
```

然鹅事实却是....5个目录！多出的那一个是什么？一个一个输出看看。

```powershell
MacBook-Pro:wuya eleme$ git cat-file -p 9dae
test1
MacBook-Pro:wuya eleme$ git cat-file -p 180c
test2
MacBook-Pro:wuya eleme$ git cat-file -p 21d0
100644 blob 180cf8328022becee9aaa2577a8f84ea2b9f3827	b.md
MacBook-Pro:wuya eleme$ git cat-file -p a16b
tree a76c93bb75184ef4b34c88a301c2351ae2219407
author eleme <xxxx@qq.com> 1576979515 +0800
committer eleme <xxxx@qq.com> 1576979515 +0800

加入到代码库中，观察objects目录变化
MacBook-Pro:wuya eleme$ git cat-file -p a76c
100644 blob 9daeafb9864cf43055ae93beb0afd6c7d144bfa4	a.md
040000 tree 21d0758079bdf2c8f7514687174454c804eb0c74	test
```

整理一下各自类型：

- 9dae-blob
- 180c-blob
- 21d0-tree
- a16b-commit
- a76c-tree

仔细一想其实也就通了，两个tree是**git根目录和test目录**。
可以得出这样一个结论：每一次commit，都会生成与之对应的commit hash值。查看历史commit也很容易得出这个结论：

![img](https://pic2.zhimg.com/80/v2-bc51d779f84b5bb0c0078d4ceb28efc9_720w.jpg)

扩展阅读：
[https://mp.weixin.qq.com/s/d4WA02Y22gdWRbmmwfPEHQ](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/d4WA02Y22gdWRbmmwfPEHQ)

## 三、Git分支

### **3.1 初探Git分支**

在学习Git分支之前，还是从git的目录树入手。

```powershell
MacBook-Pro:wuya eleme$ tree -a .git
.git
├── ......
├── HEAD
└── refs
    ├── heads
    │   └── master
    ├── remotes
    │   └── origin
    │       └── HEAD
    └── tags
```

不难看出refs目录就是用来记录当前对分支的引用信息，包括**本地分支，远程分支，标签**。

heads记录的是本地所有分支，remotes和HEAD一样，指向**对应的某个远程分支**。

```powershell
MacBook-Pro:wuya eleme$ cat .git/refs/heads/master
a16b5382a9b646a7df8d21301391f29b2f7bfb65
```

细心些就会发现，这个hash值就是**commit节点的hash值**。

而**HEAD**就是存储当前在哪个本地分支。查看其内容，可以发现：

```powershell
MacBook-Pro:.git eleme$ cat HEAD
ref: refs/heads/master
```

也就意味着，我们在本地的master上。除此之外，还可以通过`git branch`来创建其他分支。

```powershell
MacBook-Pro:.git eleme$ git branch feature/dev
MacBook-Pro:.git eleme$ git branch feature/wuya
```

切换到其他分支并查看分支信息：

```powershell
elemedeMacBook-Pro:wuya eleme$ git checkout feature/dev
切换到分支 'feature/dev'
elemedeMacBook-Pro:wuya eleme$ git branch -vv
* feature/dev  a16b538 加入到代码库中，观察objects目录变化
  feature/wuya a16b538 加入到代码库中，观察objects目录变化
  master       a16b538 加入到代码库中，观察objects目录变化
```

因此可知分支当前的指针指向最近一次commit的节点。**通过谁创建的分支，就沿用谁的指针**。注：未被放入代码库的文件会在分支切换时被抛弃，造成严重后果。

### **3.2 分支的合并**

分支的合并有两种方式，**merge**和**rebase**。

相同点：都是从一个分支获取并合并到当前分支。

**merge**：自动创建一个新的commit，如果遇到冲突，仅需要修改后重新commit。

![img](https://pic1.zhimg.com/80/v2-009bf3dd1eb907bb84aa4970d4b7afd4_720w.jpg)

每次都记录了真实详细的commit，但是在commit频繁的时候，会看到分支比较乱。比如这样，全是merge产生的节点：

![img](https://pic2.zhimg.com/80/v2-221ac1d2610b17994333c00e8943c2a5_720w.jpg)

**rebase**：找公共的节点，直接合并之前commit历史。

![img](https://pic2.zhimg.com/80/v2-d7029972815230a42ac73c108d19f67d_720w.jpg)

这样能得到简洁的分支发展历史，去掉了merge commit。但是如果合并时出现了问题，没有留下痕迹，不好定位。

- git rebase --abort：遇到冲突时放弃合并，回到rebase操作之前的状态。
- git rebase --continue：合并冲突，结合"git add 文件"命令一起，一步一步地解决冲突。
- git rebase --skip：将引起冲突的commits丢弃掉。

**小例子**

这里引用一个网上归纳的git rebase工作流：

```powershell
git rebase 
while(存在冲突) {
    //找到当前冲突文件，编辑解决冲突
    git status
    git add -u
    git rebase --continue
    if( git rebase --abort )
        break; 
}
```

注：最好不要在公共分支上使用rebase，如果前后基本上不会有别人改动你的分支，那么推荐rebase。

扩展阅读：
[https://blog.csdn.net/chenansic/article/details/44122107](https://link.zhihu.com/?target=https%3A//blog.csdn.net/chenansic/article/details/44122107)

### **3.3 分支的冲突**

**冲突的产生**

冲突是从合并的时候产生的。git分支的合并，其实就是**tree和tree的合并**。我们在feature/dev上执行`git merge master`时。**git会先找到这两个分支是从哪个指针创建出来的，称之为“merge base”。然后检查这两次的tree是否一致，如果不一致说明一定有文件发生了修改。**接下来，对于某一个文件来说，分几种情况：

![img](https://pic1.zhimg.com/80/v2-bf2e0c9a245eaf25bf06cbad6ba086bc_720w.jpg)

1. 文件在节点6，节点3，merge base中的hash值**都相同**。说明没有被修改过。不会有冲突。
2. 文件在**节点6和merge base**或者**节点3和merge base**的hash值相同时，此时直接更新文件的变化。
3. 文件在节点6，merge request，master上的hash值**都不同**，冲突就产生了。

此时就需要开发人员商定，解决冲突。

## 四、版本的回滚

如果想要版本回退，就离不开reset和revert。

**4.1 revert**

这个就一目了然了，执行`git revert`后，将回退到上一个commit的版本。

**4.2 reset**

> 前段时间，线上出了好多空指针的bug，当我查看日志定位到某一代码行时，发现该行定位不到对应的方法中。这时候就必须**切换到线上的代码版本**进行排查了。

![img](https://pic3.zhimg.com/80/v2-c05d50580dfcd5fa3f013131f4fdf26e_720w.jpg)

**git reset**分为三种模式：

- soft
- mixed
- hard

由于每一次的commit都会产生与之对应的hash值，所以借助这个进行重置就轻松多了。

**git reset --hard commit的hash值**
会重置暂存区和工作区，完全重置为指定的commit节点。当前分支没有commit的代码**必然会被清除**。

**git reset --soft commit的hash值**
会保留工作目录，并把指定的commit节点与当前分支的差异**都存入暂存区**。也就是说，没有被commit的代码也能够保留下来。

**git reset commit的hash值**
不带参数，也就是mixed模式。将会保留工作目录，并且把工作区，暂存区以及与reset的差异**都放到工作区，然后清空暂存区**。因此执行后，只要有所差异，文件都会变成红色，变得难以区分。

一般情况下，我们使用soft模式，既能保留暂存区，又能reset到某个分支。

## 五、代码暂存

当我们在当前分支工作时，不得已需要切换到其他分支处理事情而不想commit时（如果commit多了，会污染log），可以使用`git stash `将那些数据都暂存到Git提供的栈中。用法很简单～

**git stash**
暂存修改过的代码，保存在**Git栈**中，然后将工作区还原成上一次commit的内容。

```powershell
MacBook-Pro:young eleme$ git stash
保存工作目录和索引状态 WIP on wuya: 82371a5 上一次commit写的message
```

**git stash list**
显示之前压栈的所有记录。

```powershell
MacBook-Pro:young eleme$ git stash list
stash@{0}: WIP on aaa: 82371a5 上一次commit写的message
```

**git stash clear**
清空Git栈。

**git stash apply**
从Git栈中读取上一次暂存的那些代码，恢复工作区。

```powershell
MacBook-Pro:young eleme$ git stash apply
位于分支 wuya
您的分支与上游分支 'origin/wuya' 一致。

尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git checkout -- <文件>..." 丢弃工作区的改动）

        修改：     src/main/java/com/young/test/test1.java

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
```

## 最后

希望大家的Git越用越6，如有疑问，欢迎评论。
然后...卑微地求一波赞🥺

参考文章：
[https://juejin.im/post/5b6c4eeff265da0f4d0da3fa](https://link.zhihu.com/?target=https%3A//juejin.im/post/5b6c4eeff265da0f4d0da3fa)
[https://www.runoob.com/git/git-workspace-index-repo.html](https://link.zhihu.com/?target=https%3A//www.runoob.com/git/git-workspace-index-repo.html)
[https://mp.weixin.qq.com/s/d4WA02Y22gdWRbmmwfPEHQ](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/d4WA02Y22gdWRbmmwfPEHQ)
[https://blog.csdn.net/chenansic/article/details/44122107](https://link.zhihu.com/?target=https%3A//blog.csdn.net/chenansic/article/details/44122107)
https://zhuanlan.zhihu.com/p/96631135

编辑于 2020-01-10

「真诚赞赏，手留余香」

赞赏

2 人已赞赏

[![赞赏用户](https://pic4.zhimg.com/da8e974dc_is.jpg)](https://www.zhihu.com/people/yang-yun-sheng-26)[![赞赏用户](https://pic1.zhimg.com/v2-e62a104c7115a03f2a3e3bbc6dab16ce_is.jpg)](https://www.zhihu.com/people/811211534)

[Git](https://www.zhihu.com/topic/19557710)

[版本控制系统](https://www.zhihu.com/topic/19551033)

[版本管理](https://www.zhihu.com/topic/19566741)

已赞同 157140 条评论

分享

收藏



已赞同 1571

分享

### 文章被以下专栏收录

- [![Java修仙道路](https://pic4.zhimg.com/v2-4c89749e9c314f734afcf214d2112c47_xs.jpg)](https://zhuanlan.zhihu.com/c_1087293936418197504)

- ## [Java修仙道路](https://zhuanlan.zhihu.com/c_1087293936418197504)

- 记录小菜鸡的学习点滴，蟹蟹大佬们捧场！

- 关注专栏

- [![Java面试大全](https://pic2.zhimg.com/v2-d8a671c4f12c52486374d22f13075171_xs.jpg)](https://zhuanlan.zhihu.com/c_1035883039112572928)

- ## [Java面试大全](https://zhuanlan.zhihu.com/c_1035883039112572928)

- Java自古以来作为流行度最高的语言之一，在各行各业都有出彩的表现，作为一个从业者学好Java并理解他的内部机理至为重要。所以关注我们吧，你会在这使你的技能等到进一步的提示。

- 关注专栏

- [![JAVA进阶之路](https://pic2.zhimg.com/v2-f727f49083521209231b99dbf2dd5b9b_xs.jpg)](https://zhuanlan.zhihu.com/c_1081593244407640064)

- ## [JAVA进阶之路](https://zhuanlan.zhihu.com/c_1081593244407640064)

- 本专栏主要是为了帮助更多的程序员们，欢迎各位投稿，免费架构资源和面试专题资料加QQ群908676731免费获取

- 关注专栏

### 推荐阅读

- # Git 工作流

- 本文是我使用 Git 一段时间和看过一些资料后的总结，以及个人见解，深感 Git 的规范使用非常重要，不规范的使用会带来很多麻烦。 同类工具比较SVN与Git原理上Git直接记录文件快照，SVN每次…

- 五更琉璃

- # Git常用命令详解

- Git简介Git是Linux之父Linus的第二个伟大的作品，它最早是在Linux上开发的，被用来管理Linux核心的源代码。后来慢慢地有人将其移植到了Unix、Windows、Max OS等操作系统中。Git是一个分布式…

- 夏朗发表于Windy...

- ![40分钟git从入门到放弃(苦笑)](https://pic2.zhimg.com/v2-358ba38946f7aeb9f6c65999df8e050a_250x0.jpg)

- # 40分钟git从入门到放弃(苦笑)

- Lulus

- # 珍藏多年的 Git 问题和操作清单

- 引言 本文整理自工作多年以来遇到的所有 Git 问题汇总，之前都是遗忘的时候去看一遍操作，这次重新整理了一下，发出来方便大家收藏以及需要的时候查找答案。 一、必备知识点 仓库 Remote:远…

- Bug-成...发表于版本控制器

