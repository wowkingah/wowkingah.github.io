---
layout: post
title: macOS 下 Terminal 小技巧
date: 2018-03-23
tags: macOS
---

### `vim` 语法高亮  
```bash
MacBook-Pro:~ wowking$ cat ~/.vimrc
colorscheme desert
syntax on
```

### `ls` 目录等显示颜色
```bash
MacBook-Pro:~ wowking$ cat ~/.bash_profile
## set color
export CLICOLOR=1
export LSCOLORS=gxfxaxdxcxegedabagacad
MacBook-Pro:~ wowking$ source ~/.bash_profile
MacBook-Pro:~ wowking$ ls
```
CLICOLOR：是用来设置是否进行颜色的显示。CLI是Command Line Interface的缩写。  
LSCOLORS：是用来设置当CLICOLOR被启用后，各种文件类型的颜色。LSCOLORS的值中每两个字母为一组，分别设置某个文件类型的文字颜色和背景颜色。  

LSCOLORS中一共11组颜色设置，按照先后顺序，分别对以下的文件类型进行设置：  
- directory
- symbolic link
- socket
- pipe
- executable
- block special
- character special
- executable with setuid bit set
- executable with setgid bit set
- directory writable to others, with sticky bit
- directory writable to others, without sticky bit

LSCOLORS中，字母代表的颜色如下：  
- a：黑色 
- b：红色 
- c：绿色 
- d：棕色 
- e：蓝色 
- f：洋红色 
- g：青色 
- h：浅灰色 
- A：黑色粗体 
- B：红色粗体 
- C：绿色粗体 
- D：棕色粗体 
- E：蓝色粗体 
- F：洋红色粗体 
- G：青色粗体 
- H：浅灰色粗体 
- x：系统默认颜色

#### tips  
如果把目录显示成蓝色粗体，把LSCOLORS设置为Exfxaxdxcxegedabagacad就可以了。  


