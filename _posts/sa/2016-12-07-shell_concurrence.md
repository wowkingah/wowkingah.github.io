---
layout: post
title: SHELL实现多线程并发
date: 2016-12-07
tags: Shell
---

SHELL本身是按顺序执行，并没有并发概念，但可以通过几种常见方式来模拟实现多线程并发。

### 原始脚本
```bash
[root@ZTEST-218 ~]# cat task.sh
#!/bin/bash

START_TIME=`date +%s`

for ((I=1;I<=10;I++))   #模拟10个任务
do
    echo "${I}: success" ; sleep 2
done

STOP_TIME=`date +%s`
echo "TIME: $(expr ${STOP_TIME} - ${START_TIME})"

[root@ZTEST-218 ~]# sh task.s
1: success
2: success
3: success
4: success
5: success
6: success
7: success
8: success
9: success
10: success
TIME: 20
```
> tips:
> 顺序执行需要20s.


### 方式一：后台执行
```bash
[root@ZTEST-218 ~]# cat d_task.sh
#!/bin/bash

START_TIME=`date +%s`

for ((I=1;I<=10;I++))
do
    {
        echo "${I}: success" ; sleep 2
    }&
done
wait

STOP_TIME=`date +%s`
echo "TIME: $(expr ${STOP_TIME} - ${START_TIME})"

[root@ZTEST-218 ~]# sh d_task.sh
7: success
9: success
3: success
10: success
5: success
6: success
1: success
2: success
4: success
8: success
TIME: 2
```
> ***tips:***
> 本次执行时间为2s.
> 使用“{}”将执行程序变为一个块，然后用“&”将放入后台。10次执行全部放入后台后，需要用wait来等待所有后台进程执行结束，否则操作系统不会等待，而直接继续执行后续指令，直到整个程序结束。
> 缺陷：该方法不能控制运行在后台的进程数，任务较多时会拖垮操作系统。


### 方式二：管道+文件描述符
```bash
[root@ZTEST-218 ~]# cat p_task.sh
#!/bin/bash
THREAD_NUM=5
TMP_FILE="/tmp/$$.fifo"

trap "exec 6>&-;exec 6<&-;exit 0" 2     #脚本运行过程中，如果接收到信号2(Ctrl+C)中断命令，则关闭文件描述符6的读写，并正常退出

mkfifo ${TMP_FILE}  #创建有名管道
exec 6<>${TMP_FILE} #创建文件描述符，文件描述符可使用3-(n-1)，n取值范围：ulimit -n。以读写(<,读；>,写)方式绑定TMP_FILE管道文件。标识对文件描述符6的所有操作等同于对管道文件TMP_FILE的操作
rm -rf ${TMP_FILE}  #为什么不直接使用管道文件？因为管道的一个重要特性：读写必须同时存在，缺失某个操作，另一个操作就会滞留。绑定文件描述符（读、写绑定）正好解决了这个问题

for ((J=1;J<=${THREAD_NUM};J++)) #向管道中中输入THREAD_NUM个并发数量的空行。为什么写入空行而不是字符？那是因为管道文件的读取是以行为单位。
do
    echo >&6
done

START_TIME=`date +%s`

for ((I=1;I<=10;I++))
do 
    read -u6 #从管道中读取行，每次读一行。每读一次就会减少一个空行，直到管道中没有回车符，所有行读取完毕后执行挂起，实现线程数量控制。
    {
        echo "${I}: success" ; sleep 2
        echo >&6    #任务在后台执行结束后，向文件描述符中写入一个空行。如果不在向描述符中写入空行，当后台放入THREAD_NUM个任务之后，由于描述符中没有可读取的空行，会导致read -u6停顿。
    }&
done
wait

STOP_TIME=`date +%s`
echo "TIME: $(expr ${STOP_TIME} - ${START_TIME})"
 
exec 6>&-
exec 6<&-

[root@ZTEST-218 ~]# sh p_task.sh
1: success
2: success
3: success
5: success
4: success
6: success
7: success
9: success
8: success
10: success
TIME: 4
```
> ***tips:***
> 本次执行时间为4s.
  
 

### reference
http://www.jianshu.com/p/2d60e6513fdd
http://www.cnblogs.com/yxzfscg/p/5330136.html
