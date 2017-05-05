---
layout: post
title: VMware安装macOS Sierra 10.12.4
date: 2017-05-05 
tags: Mac
---

### 简述
突然之间又想倒腾一把MacOS，懒得找驱动，于是就在VMWare下开工，听闻10.12.4发布了~_~  

### 准备软件
#### securable
检测宿主机CPU是否支持虚拟化。
[securable官网](https://securable.en.softonic.com)

#### unlocker
VMWare 默认不支持安装 macOS 虚拟机，可通过 unlocker 来解锁。
[送你去百度云](http://pan.baidu.com/s/1c2hAjPe)
密码：r9fr

#### VMWare
VMware Workstation 12.5 Player 是VMWare Workstation的精简版，重要的是个人免费。
[VMware官网](http://www.vmware.com/products/player/playerpro-evaluation.html)

#### MacOS
网上清一色都是用的别人定制的虚拟硬盘，不放心，放弃。  
用ISO工具转换原生DMG镜像，试了好几次都成功，也放弃。  
懒得折腾，花了10块大洋买了个CDR(macOS 10.12.4.cdr)~_~ 
[送你去百度云](http://pan.baidu.com/s/1i4Jd3OL)
密码：i74b

### 开工
#### 1. securable
运行securable，检测CPU是否支持虚拟化。  
如下图显示，若不为YES，自行解决。
![](/images/posts/sa/install_macos_for_vmware/securable.jpg)

#### 2. VMWare
下载安装VMware Workstation Player，也可以用VMware Workstation，只要你开心。  
这时候VMWare新建虚拟机的时候，操作系统没有MAC OS类型可选，所以需要通过unlocker先解锁。

#### 3. unlocker
>3.1. 停止vmware相关服务；
> > a.windows下运行services.msc：
> > ![](/images/posts/sa/install_macos_for_vmware/services_1.jpg)
> > b.停止服务：
> > ![](/images/posts/sa/install_macos_for_vmware/services_2.jpg)
>3.2. 下载unlocker并解压；
>3.3. 以管理员身份运行。
> ![](/images/posts/sa/install_macos_for_vmware/unlocker.jpg)

#### 4. 新建虚拟机
记下图二中虚拟机名称与位置，之后要使用。
![](/images/posts/sa/install_macos_for_vmware/create_vm_1.jpg)
![](/images/posts/sa/install_macos_for_vmware/create_vm_2.jpg)
![](/images/posts/sa/install_macos_for_vmware/create_vm_3.jpg)

#### 5. 启动虚拟机
这时候启动虚拟机会报错，如下图：
![](/images/posts/sa/install_macos_for_vmware/start_vm_1.jpg)
修改虚拟机参数可解决。  
> 5.1 进入虚拟机文件夹，找到后缀为Mac OS Sierra.vmx的文件（虚拟机文件夹参照新建虚拟机图二；Mac OS Sierra为虚拟机名）：
> ![](/images/posts/sa/install_macos_for_vmware/vmx_1.jpg)
> 5.2 以记事本打开，在smc.present = "TRUE" 添加一行`smc.version = 0`
> ![](/images/posts/sa/install_macos_for_vmware/vmx_2.jpg)

#### 6. 又一次启动虚拟机
华丽的成功了。
![](/images/posts/sa/install_macos_for_vmware/install_macos_1.jpg)

#### 7. 系统安装
> 7.1 实用工具 => 磁盘工具
![](/images/posts/sa/install_macos_for_vmware/install_macos_2.jpg)
> 7.2 选择虚拟磁盘 => 抹除
> ![](/images/posts/sa/install_macos_for_vmware/install_macos_3.jpg)
> ![](/images/posts/sa/install_macos_for_vmware/install_macos_4.jpg)
> ![](/images/posts/sa/install_macos_for_vmware/install_macos_5.jpg)
> 7.3 退出磁盘工具
> ![](/images/posts/sa/install_macos_for_vmware/install_macos_6.jpg)
> 7.4 选择安装磁盘，然后一路向北
> ![](/images/posts/sa/install_macos_for_vmware/install_macos_7.jpg)
> ![](/images/posts/sa/install_macos_for_vmware/install_macos_8.jpg)
> ![](/images/posts/sa/install_macos_for_vmware/install_macos_9.jpg)
> ![](/images/posts/sa/install_macos_for_vmware/install_macos_10.jpg)


