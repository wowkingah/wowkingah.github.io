---
layout: post
title: git更新fork的repository
date: 2017-04-22
tags: git
---

Fork一个别人的repository，做了一些改动，想提交pull request的时候，发现原先别人的repository已经又有了一些更新了，这个时候想使得自己fork出的repository也得到这些更新，即和原repository同步，该怎么做呢？这个问题应该被问烂了，stackoverflow上也有解答，基本上是指向的GitHub上的官方文档。最主要的是这2篇：
https://help.github.com/articles/configuring-a-remote-for-a-fork/
https://help.github.com/articles/syncing-a-fork/
本文略作翻译，以中文解答这个问题。
------  

### 查看当前配置
首先，检查一下当前的配置，看看当前有没有已经设置了上游，这要使用git remote -v 命令。如下：
```bash
$git remote -v  
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)  
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push) 
``` 
以上表明，origin这个repository对应的是远端的https开头的这个链接指向的repository，即自己fork出的repository。

### 设置上游
第二步，将原repository设置为自己fork出的repository的上游（upstream）。运用如下的命令：
```bash
$git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```
  
运用第一步中提到的git remove -v命令再次检查一下，结果如下：
```bash
$git remote -v  
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)  
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)  
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)  
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)  
```
   
### 获取上游更新
第三步，运行 git fetch upstream 命令，如下：
```bash
$git fetch upstream  
remote: Counting objects: 75, done.  
remote: Compressing objects: 100% (53/53), done.  
remote: Total 62 (delta 27), reused 44 (delta 9)  
Unpacking objects: 100% (62/62), done.  
From https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY  
 * [new branch]      master     -> upstream/master
 ```  
以上表明，远程的原repository上确实有一些更新，现在它们已经被download到本地的.git文件夹下了，但是还没有合并到本地的代码中。

### checkout本地repo
第四步，git checkout master，这是保证切换到本地的repository的master上，如果本来就在，那么这一步不是必须的。
```bash
$git fetch upstream 
```

### merge上游更新
第五步，运行 git merge upstream/master 命令，将upstream/master上的更新合并到本地的master上，其实就是将第三步中download到.git文件夹下的那些change合并到本地的master中。如下：
```bash
$git merge upstream/master  
Updating a422352..5fdff0f  
Fast-forward  
 README                    |    9 -------  
 README.md                 |    7 ++++++  
 2 files changed, 7 insertions(+), 9 deletions(-)  
 delete mode 100644 README  
 create mode 100644 README.md  
```
如果本地没有什么自己独立的更新的话，那么将执行"fast-forward"的合并。如果本地有自己独立的更新，而又会引起冲突的话，则要解决冲突，再commit。

关于解决冲突，如果明确所有冲突都是使用upstream/master上的来override自己的，那么可以直接运行如下命令，则无需解决冲突了：
```bash
git merge -X theirs upstream/master  
```

注意，以上步骤结束后，仅仅是本地的fork出的repository和原repository取得了同步，如果想让远程的fork出的repository也同样取得同步，必须再git push上去。
 

   
### reference
http://blog.csdn.net/nirendao/article/details/51464191
