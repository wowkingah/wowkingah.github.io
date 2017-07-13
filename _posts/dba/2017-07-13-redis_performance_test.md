---
layout: post
title: Redis性能测试
date: 2017-07-13
tags: Redis VM Docker
---

### 简述  
简单压一把Redis在VM和Docker下性能如何。  
  
### 环境  
**VM**  
CPU: 2C  
Mem: 2G  
OS: CentOS release 6.7 (Final)  
 
**Docker**  
CPU: 2C  
Mem: 2G  
OS: CentOS release 6.9 (Final)  

### 测试基准值  
>* tools: redis-benchmark  
>* type: SET、GET  
>* requests: 5000000  
>* keyspacelen: 500000  
>* size: 2(default)  
>* parallel: 8 ~ 4096  
   
### 测试结果  
#### SET  
![](/images/posts/dba/redis_performance_test/qps_set.png)  

#### GET  
![](/images/posts/dba/redis_performance_test/qps_get.png)  

### 测试结论  
由于Docker很多内核参数不能调整，该测试结果建立在环境尽量一致的前提下。  
从上图测试结果看，并发数小于256时，VM读写QPS要高于Docker；并发数大于512后，Docker读写QPS要高于VM。  

### 其它  
-P: 使用管道，默认值"1"。
Redis是一种基于客户端-服务端模型以及请求响应协议的TCP服务。这意味着通常情况下一个请求会遵循以下步骤：  
>* 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。  
>* 服务端处理命令，并将结果返回给客户端。  

Redis管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。管道技术最显著的优势是提高了redis服务的性能。  

-r：不是request，而是random。后面跟的一个正整数M的含义是：0 ~ M之间的一个随机数，例如：-r 10，意思是随机0~9之间的一个数字。  

PING_INLINE和PING_BULK  
在统一请求协议之前，Redis使用一个不同的协议来发送命令，这个协议仍然被支持因为通过telnet它很容易手写。在这个协议中存在两种命令：  
> 1. ?inline命令:命令很简单，就是用空格把参数分隔开来的字符串。二进制安全是不可能的。
> 2. ?bulk命令: bulk命令和inline命令几乎是一样的，但是最后一个参数为了能够接受二进制安全的内容，所以需要以特殊的方式进行处理。  

