## 主从复制
> 进行配置
```java
1、进行配置文件拷贝
cd /usr/local/redis/conf/
cp redis.conf redis-6380.conf
cp redis.conf redis-6381.conf

2、修改6380与6381配置文件
vim redis-6380.conf
    
port 6380		# 端口
daemonize yes	# 后台启动
pidfile /usr/data/redis/run/redis_6380.pid		# 进程
logfile "/usr/data/redis/logs/redis_6380.log"	# 日志
replicaof 127.0.0.1 6379   	# master地址 
masterauth 123456    		# master密码
    
3、启动redis服务
/usr/local/redis/bin/redis-server /usr/local/redis/conf/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/conf/redis-6380.conf
/usr/local/redis/bin/redis-server /usr/local/redis/conf/redis-6381.conf
ps -ef|grep redis 
```

> 主从信息查看

INFO replication

```java
root@iZ2ze8gkwfpqlsfl8j4o14Z:~# redis-cli -p 6379
127.0.0.1:6379> AUTH 123456
OK
127.0.0.1:6379> INFO replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=462,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=462,lag=1
master_replid:b1db71d06209809ba7922894ec26e7ad14bbcc8a
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:462
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:462
```

> 注意

* 此时master可以进行读和写，而slave只能进行读不能进行写，否则会包错：

```
127.0.0.1:6381> set age 10
(error) READONLY You can't write against a read only slave # 不能对只读从机进行写操作	
```

* 如果此时主机宕机，主从不会自动切换，从机还连着主机，只能读不能写，会造成单点故障

## 哨兵模式

​	传统的主从设计，主机和从机不会自动切换，如果主机宕机，整个redis集群将无法提供服务。为了解决这个问题，redis采用了一种重新选举机制 - 哨兵机制。

​	所以redis进程主机上除了要运行redis服务进程外，还需要提供一个哨兵进程，而后所有的哨兵要求彼此通信。

当哨兵监测到mster宕机后，会自动从slave切换到master，然后通过发布订阅模式，通知其他redis服务器，修改配置文件，让他们切换主机。

​	除了监控redis服务之外，哨兵也会相互监控。如果哨兵1监测到主机宕机，不会立即故障转移，因为仅仅是哨兵1监测到，这种现象称为**主观下线**。当其他哨兵也监测到，达到一定数量时，会由其中一个哨兵发起投票，进行故障转移操作。切换成功后，会通过发布订阅模式，让各个哨兵所监控的服务器切换主机，这个过程被称为**客观下线**。

```java
1、哨兵启动文件拷贝
cp /usr/local/src/redis/src/redis-sentinel /usr/local/redis/bin/

2、哨兵配置文件拷贝
cp /usr/local/src/redis/sentinel.conf /usr/local/redis/conf/

3、哨兵信息存储目录
mkdir -p /usr/data/redis/sentinel

4、修改配置文件
vim /usr/local/redis/conf/sentinel.conf 

# 关闭redis保护模式，不关闭将无法重新选举master，无法通过哨兵修改redis.donf
protected-mode no
# 哨兵服务监听端口    
port 26379
# 是否为后台运行    
daemonize no
# 哨兵进程的的目录    
pidfile /usr/data/redis/run/redis-sentinel.pid	
# 哨兵日志    
# logfile "/usr/data/redis/logs/sentinel_26379.log"	
# 哨兵数据存储目录    
dir /usr/data/redis/sentinel	
# 设置哨兵名称、master主机地址、端口号、2表示如果有两个哨兵发现有问题，则进行重新选举    
sentinel monitor mymaster 127.0.0.1 6379 2	
# 设置redis主机认证信息    
sentinel auth-pass mymaster 123456	
# 如果哨兵在10秒内不活动，则仍为宕机    
sentinel down-after-milliseconds mymaster 10000	
# 定义master的同步数量    
sentinel parallel-syncs mymaster 1	
# 哨兵如果在5秒内没有选举成功，则认为选举失败    
sentinel failover-timeout mymaster 5000	

5、进行哨兵集群配置
cp /usr/local/redis/conf/sentinel.conf /usr/local/redis/conf/sentinel-26380.conf
cp /usr/local/redis/conf/sentinel.conf /usr/local/redis/conf/sentinel-26381.conf
mkdir -p /usr/data/redis/sentinel-26380
mkdir -p /usr/data/redis/sentinel-26381
根据第四步进行配置文件修改
vim /usr/local/redis/conf/sentinel-26380.conf
vim /usr/local/redis/conf/sentinel-26381.conf    
    
6、检查主从复制
/usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 6379 -a 123456 info replication

7、启动哨兵进程
/usr/local/redis/bin/redis-sentinel /usr/local/redis/conf/sentinel.conf 
/usr/local/redis/bin/redis-sentinel /usr/local/redis/conf/sentinel-26380.conf
/usr/local/redis/bin/redis-sentinel /usr/local/redis/conf/sentinel-26381.conf
    
8、查看sentinel状态：
root@iZ2ze8gkwfpqlsfl8j4o14Z:~# redis-cli -h 127.0.0.1 -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3    
```

> 控制台信息

```
Sentinel ID is c1bb0def8bf278fd186e4d45b67d78f36cbc1c51
+monitor master mymaster 127.0.0.1 6379 quorum 2
+sdown slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
+sdown slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
+sentinel sentinel e9dfc3f9802b4a05fea7ab0b073f520458c48943 127.0.0.1 26380 @ mymaster 127.0.0.1 6379
+sentinel sentinel b4aaecc22eb47dc69a6b6a1629c9dfd96aea1fa6 127.0.0.1 26381 @ mymaster 127.0.0.1 6379
```

