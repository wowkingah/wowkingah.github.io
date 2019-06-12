---
layout: post
title: 同一台电脑上使用多个 GIT 账号
date: 2018-08-16
tags: Git
---

### 简述  
同一台电脑上，可能需要同时使用多个 `GIT` 账号，以便管理不同的项目。比如：个人 `GIT` 账号、公司 `GIT` 账号，等等。  

### 全局账号
平时配置 `GIT` 时，一般会这么设置：  
```shell
MacBook-Pro:~ wowking$ git config --global user.name "your_name"
MacBook-Pro:~ wowking$ git config --global user.email "your_email"
# 查看全局配置
MacBook-Pro:~ wowking$ git config --global --list
```
如果一台电脑上只有一个 `GIT` 账号，使用全局账号配置自然没问题。  
一台电脑想要同时使用多个 `GIT` 账号，就需要为该项目配置本地账号。  

### 配置本地账号  
```shell
MacBook-Pro:~ wowking$ git config user.name "your_name"
MacBook-Pro:~ wowking$ git config user.email "your_email"
MacBook-Pro:~ wowking$ git config --global --list
```
*TIPS*  
1. 可使用 `--local` 选项，或不写;  
2. `--local` 优先级比 `--global` 高。  

### 取消全局账号  
如果想取消全局账号配置，可通过如下方式：  
```shell
MacBook-Pro:~ wowking$ git config --global --unset "your_name"
MacBook-Pro:~ wowking$ git config --global --unset "your_email"
MacBook-Pro:~ wowking$ git config --global --list
```

### reference  
https://blog.csdn.net/baidu_35738377/article/details/54580156  
