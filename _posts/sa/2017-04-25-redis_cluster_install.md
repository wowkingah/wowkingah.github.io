---
layout: post
title: Redis Cluster安装部署
date: 2017-04-25 
tags: Redis
---

### 简述
从Redis 3.x开始，一直单机的Redis终于步入Cluster大时代。  
Redis从一个单纯的NoSQL内存数据库变成了分布式NoSQL数据库，CAP模型也从CP变成了AP。  

### 架构
#### 架构图
![](/images/posts/sa/redis_cluster_install/redis_cluster_architecture.jpg)

#### 架构细节
>* 所有 redis 结点彼此互联（PING-PONG 机制），内部使用二进制协议优化传输速度和带宽（客户端与结点之间的通信还是正常的ascii协议）；
>* 结点的 fail 是通过集群中超过半数的结点检测失效时才生效；
>* 客户端与 redis 结点直连，不需要中间 proxy 层；
>* 客户端不需要连接集群所有结点，连接集群中任何一个可用结点即可（可用结点）；
>* redis-cluster 把所有的物理结点映射到[0-16383]slot 上，cluster 负责维护 node <-> slot <-> value。  

#### 选举与容错
![](/images/posts/sa/redis_cluster_install/fail.jpg)
##### 选举
>* 选举过程是集群中所有 master 参与，如果半数以上 master 节点与 master 节点通信超过 (cluster-node-timeout)，认为当前 master 节点挂掉；

##### 容错
什么时候整个集群不可用(cluster_state:fail)？
>* 如果集群任意 master 挂掉，且当前 master 没有 slave，集群进入 fail 状态，也可以理解成集群的 slot 映射[0-16383]不完成时进入 fail 状态。
> >* redis-3.0.0.rc1 加入 cluster-require-full-coverage 参数，默认关闭，打开集群兼容部分失败。
>* 如果集群超过半数以上 master 挂掉，无论是否有 slave 集群进入 fail 状态。
> >* 当集群不可用时，所有对集群的操作做都不可用，收到((error) CLUSTERDOWN The cluster is down) 错误。  

### Redis安装
#### 安装redis
Redis[官网](https://redis.io/download)下载并安装最新稳定版Redis。  

#### redis cluster配置
```bash
[root@Wowking-001 ~]# cat /etc/redis/6379.conf |grep -vE '^#|^$'
daemonize yes
pidfile /var/run/redis_6379.pid
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 0
loglevel notice
logfile /data/logs/redis/redis_6379.log
databases 16
stop-writes-on-bgsave-error yes
rdbcompression no
rdbchecksum no
dbfilename dump.rdb
dir /data/redis/data/6379
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
 cluster-enabled yes
 cluster-config-file nodes-6379.conf
 cluster-node-timeout 5000
 cluster-migration-barrier 1
 cluster-require-full-coverage no
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```
*tips*
关闭了持久化。  

### 系统相关参数调整
```bash
[root@Wowking-001 ~]# cat /etc/sysctl.conf
##Redis
vm.overcommit_memory = 1
net.core.somaxconn = 2048
net.ipv4.tcp_timestamps=1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_wmem = 8192 131072 16777216
net.ipv4.tcp_rmem = 32768 131072 16777216
[root@Wowking-001 ~]# sysctl -p
[root@Wowking-001 ~]# echo never > /sys/kernel/mm/transparent_hugepage/enabled  #禁用Linux内核特性transparent huge pages，不然日志会报WARNING
```

### 启动Redis
测试使用，在本机启3个不同端口的实例。
```bash
[root@Wowking-001 ~]# mkdir -p /data/logs/redis
[root@Wowking-001 ~]# mkdir -p /data/redis/data/6{3,4,5}79
[root@Wowking-001 ~]# redis-server /etc/redis/6379.conf
[root@Wowking-001 ~]# redis-server /etc/redis/6479.conf
[root@Wowking-001 ~]# redis-server /etc/redis/6579.conf
```

### cluster部署
#### 安装ruby及ruby的redis模块
```bash
[root@Wowking-001 ~]# yum -y install ruby rubygems
[root@Wowking-001 ~]# gem install redis
```

#### cluster初始化
```bash
[root@Wowking-001 ~]# redis-trib.rb create --replicas 0 192.168.1.1:6379 192.168.1.1:6479 192.168.1.1:6579
>>> Creating cluster
>>> Performing hash slots allocation on 3 nodes...
Using 3 masters:
192.168.1.1:6379
192.168.1.1:6479
192.168.1.1:6579
M: 5d6519a85e2795f0eeb7566031eb6be47c97efbf 192.168.1.1:6379
   slots:0-5460 (5461 slots) master
M: fc82bf9de8adba9886b0cb62ad8a99e8d4495f35 192.168.1.1:6479
   slots:5461-10922 (5462 slots) master
M: 35586851e194755ca82e660010b930987adfde6d 192.168.1.1:6579
   slots:10923-16383 (5461 slots) master
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join..
>>> Performing Cluster Check (using node 192.168.1.1:6379)
M: 5d6519a85e2795f0eeb7566031eb6be47c97efbf 192.168.1.1:6379
   slots:0-5460 (5461 slots) master
M: fc82bf9de8adba9886b0cb62ad8a99e8d4495f35 192.168.1.1:6479
   slots:5461-10922 (5462 slots) master
M: 35586851e194755ca82e660010b930987adfde6d 192.168.1.1:6579
   slots:10923-16383 (5461 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
*tips*
>* --replicas：指定复本数，这是cluster最小复本要求。推荐2个以上复本，不然没法测试failover。

#### 查看cluster状态
```bash
[root@Wowking-001 ~]# redis-trib.rb check 192.168.1.1:6379
>>> Performing Cluster Check (using node 192.168.1.1:6379)
M: 5d6519a85e2795f0eeb7566031eb6be47c97efbf 192.168.1.1:6379
   slots:0-5460 (5461 slots) master
   0 additional replica(s)
M: 35586851e194755ca82e660010b930987adfde6d 192.168.1.1:6579
   slots:10923-16383 (5461 slots) master
   0 additional replica(s)
M: fc82bf9de8adba9886b0cb62ad8a99e8d4495f35 192.168.1.1:6479
   slots:5461-10922 (5462 slots) master
   0 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### cluster测试
```bash
#任意结点写key
[root@Wowking-001 ~]# redis-cli -p 6379
127.0.0.1:6379> set aa 11
OK
127.0.0.1:6379> get aa
"11"
#任意结点get
[root@Wowking-001 ~]# redis-cli -p 6579 -c
127.0.0.1:6579> get aa
-> Redirected to slot [1180] located at 192.168.1.1:6379
"11"
```
*tips*
>* -c：指定cluster模式。


### reference
https://redis.io/topics/cluster-tutorial  
http://blog.csdn.net/dc_726/article/details/48552531  
http://www.ttlsa.com/redis/redis-cluster-theoretical-knowledge  
