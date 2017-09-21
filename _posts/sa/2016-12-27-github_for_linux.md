---
layout: post
title: Linux下使用github
date: 2016-12-27
tags: Git
---

### 1. GITHUB环境准备

#### 1.1 申请GITHUB账号：https://github.com/

#### 1.2 生成KEY
```
[wowking@ZTEST-218 git]$ ssh-keygen -t rsa -C "wowkingah@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/wowking/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/wowking/.ssh/id_rsa.
Your public key has been saved in /home/wowking/.ssh/id_rsa.pub.
The key fingerprint is:
e5:1f:b6:cc:09:e0:bd:2a:d2:bb:1d:36:b4:a8:33:70 wowkingah@gmail.com
The key's randomart image is:
+--[ RSA 2048]----+
| |
| |
| . . |
| . = |
| S + o |
| . E o . B + |
| o .. = . * |
| +.oo + |
| .+o+o |
+-----------------+
```

#### 1.3 查看生成的KEY
```
[wowking@ZTEST-218 git]$ ls ~/.ssh/
id_rsa id_rsa.pub known_hosts
[wowking@ZTEST-218 git]$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA2nkR/J0WOSrV5g1/JseP+e0Wv+VejDs+Qk1wXpbyWQd5W1QD4nJYlYaM58vpUMhooW+iC4ciA4PGwxMEwc+wXLs0JZAgKVuXW8moJCsnXfMxo0eUX1l0wWeuVgSo46tb/U+0bsQNmMhNj4UQgflDABHGWGdZqg/8nV5kGr0VDpmwPvo7gbdm+56rQDRPyldSo+sbDvS78zzidpJi+OmG4wpA7emvrQYC2T9YaJIgzNjG6Gvpbe0BG3it0/n5cIXy5PW56jhdsb1KO4A9fLNximb0y7FJluhcaoxvCGFP1OnBo4IfdiRIoEzYCS8xzRz6h7WxgQGZFjxbB3WITNpkwQ== wowkingah@gmail.com
```

#### 1.4 将公钥id_rsa.pub添加至github
github => settings => SSH and GPG keys => New SSH key => Add SSH key

#### 1.5 测试
```
[wowking@ZTEST-218 git]$ ssh -T git@github.com
The authenticity of host 'github.com (192.30.253.113)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.253.113' (RSA) to the list of known hosts.
Hi wowkingah! You've successfully authenticated, but GitHub does not provide shell access.
```


### 2. 常用GIT命令
#### 2.1 clone
```
[wowking@ZTEST-218 git]$ git clone git@github.com:wowkingah/test.git
Initialized empty Git repository in /home/wowking/git/test/.git/
warning: You appear to have cloned an empty repository.
[wowking@ZTEST-218 git]$ cd test
[wowking@ZTEST-218 test]$ echo "# test1" > README.md
[wowking@ZTEST-218 test]$ ll
total 4
-rw-rw-r-- 1 wowking wowking 8 Dec 26 16:35 README.md
```

#### 2.2 add
```
[wowking@ZTEST-218 test]$ git add README.md
```

#### 2.3 commit
```
[wowking@ZTEST-218 test]$ git commit -m "first commit"
[master (root-commit) a72396a] first commit
1 files changed, 1 insertions(+), 0 deletions(-)
create mode 100644 README.md
```

#### 2.4 push
```
[wowking@ZTEST-218 test]$ git push -u origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 217 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:wowkingah/test.git
* [new branch] master -> master
Branch master set up to track remote branch master from origin.
```
