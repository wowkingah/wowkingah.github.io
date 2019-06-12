---
layout: post
title: SVN 代码迁移至 Git
date: 2019-06-12
tags: Git
---

### 简述
以前使用 `SVN` 存放代码，现将代码迁移至 `GIT`。  
只迁移 `trunk` 分支，以及所对应的 `commits` 记录。   

## SVN checkout
```shell
MacBook-Pro:tmp wowking$ mkdir svn git
# checkout, SVN 路径为 BASE 路径
MacBook-Pro:tmp wowking$ svn --username svn_user --password svn_pwd checkout svn://127.0.0.1/test-scheduler svn/test-scheduler
```

## SVN 与 GIT 用户名映射
```shell
# 获取 SVN 项目中所有 commits 对应的用户名
MacBook-Pro:tmp wowking$ svn log svn/test-scheduler/ --xml | grep author | sort -u | perl -pe 's/.*>(.*?)<.*/$1 = /' > users.txt
MacBook-Pro:tmp wowking$ cat users.txt
admin =
zhangsan =
lisi =

# 按 Git 格式补充。等号前为 SVN 格式下的用户名，等号后为 GIT 格式下用户名与邮箱地址
$ cat users.txt
admin = Administrator <admin@xxx.com>
zhangsan = 张三 <zhangsan@xxx.com>
lisi = 李四 <lisi@xxx.com>
```

## clone from svn to git
```shell
$ git svn clone svn://127.0.0.1/test-scheduler \
--stdlayout --no-metadata --authors-file=users.txt -s git/test-scheduler
```
*tips*
> --stdlayout：表示项目在 SVN 中是常见的 “trunk/branches/tags” 目录结构。如果不是，则需要使用 --tags，--branches, --trunk 参数（语法：git svn help）  
> --no-metadata：不导出 SVN 包含的一些无用信息  
> --authors-file：SVN 账号映射到 GIT 账号对应的文件，所有 SVN 作者都要做映射  

## 分支处理  
```shell
MacBook-Pro:tmp wowking$ cd git/test-scheduler/
MacBook-Pro:test-scheduler wowking$ git branch -a
* master
  remotes/origin/dev
  remotes/origin/trunk
# 这里不迁其它分支与Tags
MacBook-Pro:test-scheduler wowking$ rm -Rf .git/refs/remotes
MacBook-Pro:test-scheduler wowking$ git branch -a
* master
```

## push
```shell
# 初始化远程仓库
MacBook-Pro:test-scheduler wowking$ git remote add origin http://git.xxx.com/git_test/test-scheduler.git
# push
MacBook-Pro:test-scheduler wowking$ git push -u origin master
remote:   http://git.xxx.com/git_test/test-scheduler
remote:
To http://git.xxx.com/git_test/test-scheduler.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```



