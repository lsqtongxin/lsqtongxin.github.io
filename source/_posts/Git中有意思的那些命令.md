---
title: Git中有意思的那些命令
date: 2021-06-29 09:19:40
tags:
---

哎呀，这是我第一篇技术文章啦，写这篇文章是为了记录一下那些被忽略的命令，另外也是想把这些内容分享给大家。

当前系统为windows7或windows10

### 1.git merge-base

命令描述：这个命令用于查找两个分支最近公共祖先，归属于查询命令，平时用的不多,其实和二叉树查找最近公共祖先的原理(leetcode 235/236)是一致的。

使用场景： 可以在三路合并merge操作之前查询两个分支或两个提交的最近公共祖先，也就是他们的分叉点。

```
git merge-base branchA branchB
git merge-base commitA commitB

举例：查询远程dev分支和本地dev分支的最近公共祖先
git switch dev
git fetch origin dev
git merge-base dev origin/dev
```

fork-point

### 2.git bisect

命令描述：这个命令用于查找第一个出错的版本,其实和二分查找法(leetcode278)是一致的，也是一个应用了算法的命令。

使用场景： 当目前的最新的节点是一个出错的版本，你也知道最近上线的正确版本(tag/branch/commitID),其中这些提交历史很长，可以使用这个命令进行查找；除了查找错误节点，还可以查找其他具有某些属性的节点，以old/new来代替good/bad。


```
git bisect start
git bisect bad
git bisect good <commitID> or <tag>
git bisect good
git bisect bad
git bisect reset 
```

感觉这个命令非常好玩。

### 3.git stash

命令描述：这个命令用于记录工作目录和缓存区的当前状态，并保护现场恢复现场，是一个应用堆栈数据结构的命令。

使用场景： 在有修改文件的工作目录拉取更新，在工作中被其他需求打断，测试部分的提交

```
git stash push						//进行存储，其实与 git stash作用是等效的
git stash list						//查看堆栈列表
git stash show 						//查看修改信息
git stash pop						//堆栈弹出
git stash drop stash@{n}			//删除某一个stash@{n}
git stash apply stash@{n}		 	//应用某一个stash@{n}
```
当有多次stash时，stash@{0}表示最近最新，其中stash@{n}的n越大，表示数据越旧越早，这些指令都是增删改查。
pop是按照堆栈来进行弹出，而apply是应用某一个并且也不移除这个stash。

### 4.git cherry-pick

命令描述：这个命令用于将一个分支上一个或多个提交应用到另外一个分支上，类似复制粘贴功能。可以理解为在樱桃树上摘樱桃，然后进行嫁接到其他分支，长出樱桃苹果，原谅我是一个吃货。

使用场景：和上面的命令描述一样。

```
git cherry-pick commitID                       //复制commitID，并应用粘贴到当前分支  
git cherry-pick startCommitID endCommitID      //复制(startCommitID endCommitID] 并应用粘贴到当前分支
```
**注意这个范围是左开右闭**
如果冲突，请修改完冲突，在进行add修改并git cherry-pick --continue，否则直接回退git cherry-pick --abort


```
git rebase --onto branchName startCommitID endCommitID 
```
对于范围是 (startCommitID, endCommitID] 左开右闭，这里建议写commitID，而不是分支，这样清晰准确。
使用场景：
1.仅仅rebase某个分支的部分提交，而不是全部提交  
2.切换这个分支对应的依赖节点到其他分支或节点  
3.分离不同分支的依赖关系  
4.在提交历史中剔除错误的提交

如果我没有说清楚的话，请参考官方文档https://git-scm.com/docs/git-rebase

**总结：git cherry-pick和git rebase --onto 非常相似，但cherry-pick实现的是复制粘贴，而rebase --onto实现的是剪切粘贴，其实都是果树嫁接**


### 总结

Git里面也包含了计算机的一些算法(最近公共祖先和二叉查找树)及数据结构(堆栈)，虽然比较简单，但是在开发过程中看到算法还是比较新鲜的，毕竟大家都是CURD boy。
把Git当做樱桃树，想象它挂满了樱桃，merge-base是寻找两个树枝最高的树叉，bisect是找到树枝和树干上坏了的节点，cherry-pick是摘樱桃实现复制粘贴嫁接，rebase --onto是实现剪切粘贴嫁接，stash是将当前状态裹上塑料布，再进行其他操作。 将这些命令类比于对樱桃树的操作，更易于学习Git. 学习资料可以参考后面的引用，Git官方文档和廖雪峰老师的Git中文教程。

（完）

References
[1] Git官方文档: https://git-scm.com/docs/
[2] 廖雪峰 https://www.liaoxuefeng.com/wiki/896043488029600