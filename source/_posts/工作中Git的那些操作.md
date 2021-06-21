---
title: 工作中Git的那些操作
date: 2021-06-19 09:19:40
tags:
---

其实接触Git也很长时间了，自打上学那会就用Git作为版本控制工具，感触颇深。写这篇文章也是记录一下自己的理解留作日后词典进行查询，另外也是想把这些内容分享给大家。本篇将以问题为导向来阐述相关的知识，，面对的对象为具有一定经验的开发者。

假设当前项目有两个分支，为master和dev，所有开发工作都在dev上进行，而master分支为用于发布生产代码的分支，当前系统为windows7或windows10

**我整理了我在日常工作中的十个问题，重点是第4个问题和第8个问题，如下所示：**

### 1.撤销本地工作目录中的更改

```
git checkout -- <file>
```

这样将撤销对本地工作目录的更改。

### 2.撤销本地缓存的更改

```
git reset <file>			//从缓存区移除特定文件，但不改变工作目录，与git add <file>是反义词
git reset					//它会取消 *所有* 文件的缓存，而不会覆盖任何修改，与`git add .`是反义词
git reset --hard			//它会取消 *所有* 文件的缓存，并且恢复工作目录
```

### 3.当在本地提交commit后如何修改这个commit message？

**git commit命令有一个amend参数，用于修改之前的提交信息**

```
git commit --amend
```

当点击回车后，会出现一个编辑框，重新编辑内容并保存退出即可。

其实还有一个场景是：你写的commit message是对的，但是你忘记了修改了某个内容，那怎么办？

这时候你有两个方法，第一就是重新修改然后add和commit，即生成一个新的节点。

第二种就是在当前这个节点上修改然后add，最后amend，即在原来的节点进行修改。

```
vim  test.txt

git add . 

git commit --amend
```

这是一种将忘记的修改添加到当前这个节点的方法

**总结：amend这个参数不仅仅能修改commit message，还能对修改进行更正**

### 4.当周三升级要对某个版本进行打包

首先通过`git log`命令查询当前的历史，例如：

```
commit ae4fa4a6d8f9b4f6095c822935e3e0f7ab20f7cd (HEAD -> master)
Author: lsqtongxin <lsqtongxin@qq.com>
Date:   Sat Jun 19 11:09:06 2021 +0800

    add third.java

commit 1cf533f4bca614a163810da46657efc9343075d5
Author: lsqtongxin <lsqtongxin@qq.com>
Date:   Sat Jun 19 11:08:18 2021 +0800

    add second.java

commit 9d69109d65ffd13291c3f72f304e36a20b99b32b
Author: lsqtongxin <lsqtongxin@qq.com>
Date:   Sat Jun 19 11:06:48 2021 +0800

    add main.java
```

当你想使用second.java(1cf533f)来进行周三生产升级，但是当前的最新的节点为third.java(ae4fa4a)它是你最近完成的代码内容，即你电脑硬盘里显示为third.java版本。那你如何操作呢？

使用git checkout <commitID>命令来针对某个版本进行检出代码：

```
git checkout 1cf533f              //这样你的硬盘将会恢复为second.java文件等
```

然后进行代码的打包、编译、测试即可，待工作完成即可。

**又有人问，我第二天上班时候，如何恢复到最新的状态呢？**

通过git log是没法找到third.java的commitID的，那么如何找到third.java的commitID呢？

假设我们刚刚是在master上进行操作，所以最新的third.java节点位于master分支上。

```
git checkout master                //这样将会使恢复如初。
```

**总结： `git checkout <commitID>` 本质上是通过改变HEAD的指向来进行移动节点，它的移动的颗粒度为某个commit版本 ，在多分支操作中，这个命令还用于切换分支**

### 5. 开发某个代码时想看一下之前的版本中某个文件

接上面的第4步骤中的`git log`,例如：

```
git checkout <commit> <file>
```

查看文件之前的版本。它将工作目录中的 `<file>` 文件变成 `<commit>` 中那个文件的拷贝，并将它加入缓存区。

那么如何返回最新的版本的file呢？

```
git chekcout HEAD <file>
```

**总结： 相比较于第4步骤，它的移动的颗粒度为某个commit版本的某个文件**

### 6.合并你的多个提交(压缩提交)

假设你在dev分支上，新建一个feature分支开发一个功能点，计划5个工作日完成，你为了不丢失每天的工作内容，每天在本地commit一个提交来记录你当天的工作，那么你就有了5个commitID，导致你后续会因为开发很多功能点会有很多个commitID，有的功能点需要10个工作日即10个commitID，你会发现这些commit非常繁杂。

那么我要是按照功能点的颗粒度，是否可以将这5个commitID或者10个commitID合并为一个commitID来呢？

这样会将你的历史很简洁、干净和清爽，翻阅过往的历史也会很快速。

```
git rebase -i <base>				//在feature分支上
```

其中`<base>`为一个基准点，为某一个commitID，都是以这个基准点进行合并，但是合并的内容不包括这个基准点。

```
git rebase -i cf375ac				//在feature分支上
```

假设我们的历史从古至今为：25ccabd->0134238->d46d81e，也就是合并这三个节点为一个节点。

25ccabd为feature的第一天工作

0134238为feature的第二天工作

d46d81e为feature的第三天工作

而这个基准点为cf375ac，这个commitID cf375ac内容等等均不会变化。这里一定注意参数这不是25ccabd,然后将0134238和d46d81e前面文字由pick改为squash或者s，保存退出，然后再重新写提交信息，最后保存退出即可。

### 7.拉取更新

先 fetch，然后 merge。`git pull` 命令是整合了这两个过程的方式。

```
git fetch <remote> <branch>        //从远程拉取某个分支的更新，`git fetch <remote>`会拉取所有分支
```

例如：

```
git fetch origin dev			   //拉取远程origin/dev分支的更新

git branch -r					   //查看远程分支

git checkout dev				   //切换到dev分支

git merge origin/dev			   //将远程origin/dev合并到本地dev分支上
```

于是我们得到了远程origin/dev的更新，将其他同事的commit更新到了我的本地。

`git pull`是上面两个过程的整合，可以参考第8步。

### 8.分支合并

假设我们基于dev分支开发某一个feature功能点：

```
git branch feature dev				//基于dev新建一个feature分支
git checkout feature				//切换到feature分支进行开发工作
```

当多人协作时候，有许多人都合并了某个feature分支到dev分支上，导致dev分支有很多更新。

那么当我们完成了我们的feature功能后，我们如何操作呢？

首先我们应该远程拉取dev分支的更新:

```
git checkout dev					//切换到dev分支
git pull origin 					//拉取远程origin的dev分支的更新
```

> 如果本地dev已经提交了更新，那么需要合并本地dev和远程dev：`git pull --rebase origin`

然后在feature分支上进行rebase操作：

```
git rebase -i dev					//这一步类似第6步骤
```

最后切换到dev分支进行合并：

```
git checkout dev
git merge feature
```

再进行将dev分支推送到远程主机

```
git push origin dev
```

**总结：**

**整体步骤分为第一pull：先拉取远程dev的更新，**

**第二rebase：再基于dev进行 rebase，**

**第三merge，最后将feature合并到dev分支。**

**第四是push**

### 9.撤销某次commit的更改

```
git revert <commitID>
```

这个命令的重点会再生成一个新的节点，并且不会删除这个旧的节点，这样避免了丢失项目历史。

一般用于当某个历史提交引入了bug，一般是解决这个bug并且再生成一个新的节点。或者是将那个节点revert退回。

### 10.移除某次commit到目前的更改

```
git reset <commit>
```

将当前分支的末端移到 `<commit>`，将缓存区重设到这个提交，但不改变工作目录。所有 `<commit>` 之后的更改会保留在工作目录中，这允许你用更干净、原子性的快照重新提交项目历史。可以和rebase操作类似。

```
git reset --hard <commit>
```

将当前分支的末端移到 `<commit>`，将缓存区和工作目录都重设到这个提交。它不仅清除了未提交的更改，同时还清除了 `<commit>` 之后的所有提交