## Redis概述

> redis是什么

Redis（Remote Dictionary Server），远程字典服务

​	Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

​	Redis是一个非常快速的开源非关系、Key-Value数据库，通常称为数据结构服务器；它存储了五种不同类型值（String/Hash/List/Set/ZSet）的键映射。用作数据库，缓存和消息代理。

> 能干吗

1. 内存存储，并且可持久化到磁盘（rdb，aof）可以作为数据库使用
2. 效率高可以作为高速缓存
3. 可做做消息订阅与发布
4. 可以保存验证码，tocket，session等共享信息
5. 可以做计数，浏览量

> 有什么特性

1. 支持多样的数据类型
2. 持久化
3. 集群
4. 事务

## Linux下安装与配置

> 下载

1、官网：https://redis.io/

2、下载：http://download.redis.io/releases/redis-6.0.5.tar.gz

> 安装

上传或下载安装包，进行解压安装

```java
tar xzvf /srv/ftp/redis-6.0.5.tar.gz -C /usr/local/
cd /usr/local/redis-6.0.5/
mark
cd src
make install  # 正常安装,redis的默认安装路径/usr/local/bin		
make install PREFIX=/usr/local/bin/redis-6.0.5	#因为以前安装过故此处指定前缀
```

> 配置

安装成功后进行配置并启动

1. 移动配置文件到安装目录

```
cp /usr/local/redis-6.0.5/redis.conf /usr/local/bin/redis-6.0.5/bin/
```

​	2. 修改redis为后台启动

vim redis.conf

```
daemonize yes
```

​	3. 启动redis服务

```
./redis-server redis.conf	
ps -ef|grep java

```

```
root@iZ2ze8gkwfpqlsfl8j4o14Z:/usr/local/bin/redis-6.0.5/bin# redis-server redis.conf 
*** FATAL CONFIG FILE ERROR ***
Reading the configuration file, at line 88
>>> 'protected-mode yes'
Bad directive or wrong number of arguments
```

​	因为之前安装过redis，可能有些命令添加进了redis环境变量，所以redis-server redis.conf报错，错的意思是, 本次启动指定的配置文件目录是错误的或者配置文件的参数数量不对.原因就出在, 第一次安装redis-4.0.8时, 写如了环境变量, 执行redis-server时, 会先去查询环境变量里有没有配置这条指令, 发现有(还是旧的4.0.8的). 但是使用的配置文件是5.0.7的, 于是报错;解决方法是, 使用./指定为当前目录下的redis-server, 直接指定, 不让系统查redis-server环境变量:

 5. 使用redis-cli进行连接，进行命令测试

    redis-cli -p 6379

```java
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set name lul
OK
127.0.0.1:6379> get name
"lul"
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> shutdown
not connected> exit
```

## 基础知识

#### 1、redis默认有16个数据库，在redis.conf中配置，默认使用的是第0个，可以使用select切换数据库

```java
127.0.0.1:6379[2]> dbsize	# 查看db大小
(integer) 0
127.0.0.1:6379[2]> set name ll	# 切换数据库
OK
127.0.0.1:6379[2]> get name # 通过key获取value
"ll"
127.0.0.1:6379[2]> keys *	# 查看所有key
1) "name"
127.0.0.1:6379[2]> flushdb	# 清空当前数据库
OK
127.0.0.1:6379[2]> flushall	# 清空所有数据库
OK
```

#### 2. redis是单线程的

​	Redis是基于内存操作的，CPU不是redis的瓶颈，redis的瓶颈在网络上。

* 为什么redis是单线程的：

​	以前一直有个误区，以为高性能服务器一定是用多线程来实现。原因很简单因为误区二导致的：多线程 一定比 单线程 效率高，其实不然！

>redis 核心就是 如果我的数据全都在内存里，我单线程的去操作 就是效率最高的，为什么呢，因为多线程的本质就是 CPU 模拟出来多个线程的情况，这种模拟出来的情况就有一个代价，就是上下文的切换，对于一个内存的系统来说，它没有上下文的切换就是效率最高的。redis 用 单个CPU 绑定一块内存的数据，然后针对这块内存的数据进行多次读写的时候，都是在一个CPU上完成的，所以它是单线程处理这个事。在内存的情况下，这个方案就是最佳方案 —— 阿里 沈询

## 五大基本数据类型

### Redis-Key

```java
root@iZ2ze8gkwfpqlsfl8j4o14Z:/usr/local/bin/redis-6.0.5/bin# ./redis-cli -p 6379
127.0.0.1:6379> keys *		# 查询所有key
(empty array)
127.0.0.1:6379> set name lul # 设置key-value
OK
127.0.0.1:6379> set age 1
OK
127.0.0.1:6379> exists age	# 判断key是否存在，存在返回1
(integer) 1
127.0.0.1:6379> exists ages	# 判断key是否存在，不存在返回0
(integer) 0
127.0.0.1:6379> move age 0	# 移除当前key，如果key不存在报错
(error) ERR source and destination objects are the same
127.0.0.1:6379> move age 1	# 移除当前key，成功返回1
(integer) 1
127.0.0.1:6379> expire name 1	# 设置key的过期时间，单位秒
(integer) 1
127.0.0.1:6379> type name	# 查看当前key的类型
string
```

### String（字符串）

* 基本操作

```
127.0.0.1:6379> set k1 v1	# 设置值
OK
127.0.0.1:6379> get k1	# 获取值
"v1"
127.0.0.1:6379> keys *	# 获取所有key
1) "k1"
2) "sex"
127.0.0.1:6379> exists k1	# 判断key是否存在，存在返回1
(integer) 1
127.0.0.1:6379> exists k11	# 判断key是否存在，存在返回0
(integer) 0
127.0.0.1:6379> append k1 " hello"	# 追加字符串，不存在相当于set key
(integer) 8
127.0.0.1:6379> get k1
"v1 hello"
127.0.0.1:6379> strlen k1	# 获取字符串长度
(integer) 8
```

* i++、步长 i+=

```
127.0.0.1:6379> set views 10	# 初始值
OK
127.0.0.1:6379> incr views	# 自增1，返回自增后结果
(integer) 11
127.0.0.1:6379> incr views	
(integer) 12
127.0.0.1:6379> decr views	# 自减1
(integer) 11
127.0.0.1:6379> incrby views 3	# 步长增
(integer) 14
127.0.0.1:6379> decrby view 5	# 步长减，不存在创建
(integer) -5
127.0.0.1:6379> decrby views 5
(integer) 9
```

* 字符串范围
  * getrange	# 获取范围
  * setrange 	# 替换/截取
  * setex（set with expire）	# 设置过期时间
  * setnx（set if not exist） # 不存在则设置（分布式锁中常常使用）

```java
127.0.0.1:6379> set k1 "hello woed hello"	# 设置值
OK
127.0.0.1:6379> getrange k1 0 5	# 截取字符串
"hello "
127.0.0.1:6379> getrange k1 0 -1	# 获取全部字符型==get key
"hello woed hello"
127.0.0.1:6379> set k2 java
OK
127.0.0.1:6379> setrange k2 1 mm	# 替换指定位置字符串
(integer) 4
127.0.0.1:6379> get k2
"jmma"
127.0.0.1:6379> setex k3 30 "hello"	# 设置过期时间
OK
127.0.0.1:6379> ttl k3	# 返回key的剩余时间，不存在返回-2
(integer) 22
127.0.0.1:6379> setnx k4 "redis"	# key不存在，则创建
(integer) 1
127.0.0.1:6379> ttl k31
(integer) -2
```

* mset/mget

```
127.0.0.1:6379> mset k1 v1 k2 v3	# 批量设置
OK
127.0.0.1:6379> mget k1 k2 k3	# 批量获取
1) "v1"
2) "v3"
3) (nil)
127.0.0.1:6379> msetnx a1 v1 a2 v2	# 原子性批量操作
(integer) 1
127.0.0.1:6379> keys *	# 获取所有keys
1) "a2"
2) "k2"
3) "a1"
4) "k1"
```

* getset：先get然后set

```
127.0.0.1:6379> getset db redis	# 如果不存在，则返回nil
(nil)
127.0.0.1:6379> get db	
"redis"
127.0.0.1:6379> getset db abc	# 如果存在值，返回原来的值，并设置新值
"redis"
127.0.0.1:6379> get db
"abc"
```

String除了是字符串外还可以是数学，应用场景：计数器、对象缓存存储、JSON

### List（列表）

​	在redis里面，我们可以吧List玩成栈、队列、阻塞队列等。所有list命令都以l开头，redis不区分大小写

```java
********lpush、rpush、lrange
127.0.0.1:6379> lpush list one	# 将一个或多个值插入到列表头部（左）
(integer) 1
127.0.0.1:6379> lpush list tow	
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
127.0.0.1:6379> lrange list 0 -1	# 获取list中的值
1) "three"
2) "tow"
3) "one"
127.0.0.1:6379> lrange list 0 1	# 通过区间获取具体的值
1) "three"
2) "tow"
127.0.0.1:6379> rpush list righr a	# 将一个或多个值插入到列表尾部（右）
(integer) 5
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "tow"
3) "one"
4) "righr"
5) "a
    
********lpol、rpop、lrang、lindex、llen
127.0.0.1:6379> lpop list	# 移除左边的元素
"three"
127.0.0.1:6379> rpop list	# 移除右边的元素
"a"
127.0.0.1:6379> lrange list 0 -1	# 获取key中的值
1) "tow"
2) "one"
3) "righr"
127.0.0.1:6379> lindex list 0	# 通过下标获取key中的值
"tow"
127.0.0.1:6379> llen list	# 返回列表的长度
(integer) 3    
    
    
********lrem移除指定的值
127.0.0.1:6379> lrem list 1 b	# 移除list中指定个数的value,精准匹配
(integer) 1
127.0.0.1:6379> lrem list 1 e
(integer) 1
127.0.0.1:6379> lrem list 2 c
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "tow"
2) "one"
3) "righr"
4) "a"
5) "d  

********ltrim
127.0.0.1:6379> rpush list a1
(integer) 1
127.0.0.1:6379> rpush list a2
(integer) 2
127.0.0.1:6379> rpush list a3
(integer) 3
127.0.0.1:6379> rpush list a4
(integer) 4
127.0.0.1:6379> ltrim list 1 2	# 通过下标截取指定的长度
OK
127.0.0.1:6379> lrange list 0 -1
1) "a2"
2) "a3 
********rpoppush	 
127.0.0.1:6379> lrange list 0 -1
1) "a2"
2) "a3"
127.0.0.1:6379> rpoplpush list mlist	#移除列表的最后一个元素，并将他移动到新的列表中 
"a3"
127.0.0.1:6379> lrange mlist 0 -1	查看新列表
1) "a3"
127.0.0.1:6379> lrange list 0 -1	查看原来列表
1) "a2"    
********lset
127.0.0.1:6379> exists list		# 判断key是否存在
(integer) 1
127.0.0.1:6379> lset list 0 redis	# 通过指定下标进行替换
OK
127.0.0.1:6379> lrange list 0 -1	# 查看key
1) "redis"
127.0.0.1:6379> lset list 1 v	# 如果不存在，则会报错
(error) ERR index out of range    
    
********linsert    
127.0.0.1:6379> lpush list a b c 	
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "c"
2) "b"
3) "a"
127.0.0.1:6379> linsert list after b hello	# 插入指定元素前面
(integer) 4
127.0.0.1:6379> linsert list before b word #插入指定元素后面
(integer) 5
127.0.0.1:6379> lrange list 0 -1
1) "c"
2) "word"
3) "b"
4) "hello"
5) "a"     
```

> 可以作为队列，栈，链表、数组使用

### Set（集合）

set中的值是不能重读的

```
127.0.0.1:6379> sadd set a b c		# set集合中添加值
(integer) 3
127.0.0.1:6379> smembers set		# 查看set集合
1) "c"
2) "b"
3) "a"
127.0.0.1:6379> sismember set a		# 判断set集合中是否存在
(integer) 1
127.0.0.1:6379> scard set			# 获取set集合元素个数
(integer) 3
127.0.0.1:6379> srem set b			# 移除set集合中指定元素
(integer) 1
127.0.0.1:6379> sadd set cd rf gf
(integer) 3
127.0.0.1:6379> srandmember set		# 随机抽取出一个
"cd"
127.0.0.1:6379> srandmember set 2	# 随机抽取出多个元素
1) "gf"
2) "c"
127.0.0.1:6379> spop set			# 随机删除一个元素
"gf"
127.0.0.1:6379> spop set 2			# 随机删除多个元素
1) "c"
2) "rf"
127.0.0.1:6379> smembers set		# 查看set集合
1) "cd"
2) "a"
127.0.0.1:6379> smove set mset cd	# 将一个指定的值，移动到另外一个set集合
(integer) 1
127.0.0.1:6379> keys *
1) "mset"
2) "set"
127.0.0.1:6379> sadd set a b e		# set集合添加元素
(integer) 2
127.0.0.1:6379> sadd mset b e d
(integer) 3
127.0.0.1:6379> sdiff set mset		# 差集
1) "a"
127.0.0.1:6379> sinter set mset		# 交集
1) "b"
2) "e"
127.0.0.1:6379> sunion set mset		# 并集
1) "e"
2) "d"
3) "cd"
4) "b"
5) "a"
```



### Hash（哈希）

Map集合，key-map 这个 时候value是一个map集合，本质上和String没有多大区别

```
127.0.0.1:6379> hset hash h1 v1 h2 v2	# set一个或多个map
(integer) 2
127.0.0.1:6379> hget hash h2			# 获取指定字段值
"v2"
127.0.0.1:6379> hmset myhash h1 v1 h2 v2	# set多个key-value
OK
127.0.0.1:6379> hmget myhash h1 h2		# 获取多个字段值
1) "v1"
2) "v2"
127.0.0.1:6379> hgetall hash			# 获取全部数据
1) "h1"
2) "v1"
3) "h2"
4) "v2"
127.0.0.1:6379> hdel myhash h1			# 删除hash中指定key字段
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "h2"
2) "v2"
127.0.0.1:6379> hlen hash				# 获取hash表的字段数量
(integer) 2
127.0.0.1:6379> hexists hash field		# 判断hash中指定字段是否存在
(integer) 0
```

hash更适合对象的存储，String更加适合字符串的存储

### ZSet（有序集合）

在set基础上，增加了一个值

```
127.0.0.1:6379> zadd zset 2 d 6 g 8 y 		# 添加一个或多个用户
(integer) 3
127.0.0.1:6379> zrangebyscore zset -inf +inf	# 显示全部的用户，从小到大
1) "d"
2) "g"
3) "y"
127.0.0.1:6379> zrange zset 0 -1	# 从大到小进行排序
1) "d"
2) "g"
3) "y"
127.0.0.1:6379> zrangebyscore zset -inf +inf withscores	# 显示全部的用户，并且附带成绩
1) "d"
2) "2"
3) "g"
4) "6"
5) "y"
6) "8"
127.0.0.1:6379> zrem zset g			# 删除有序集合中的指定元素
(integer) 1
127.0.0.1:6379> zrange zset 0 -1
1) "d"
2) "y"
127.0.0.1:6379> zcard zset			# 获取有序集合中的个数
(integer) 2
127.0.0.1:6379> zcount zset 1 6		# 获取指定区间的成员数量
(integer) 1
127.0.0.1:6379>
```

​	其余API可参考官方文档

## 三种特殊数据类型

### Geospatial地理位置



### Hyperloglog



### Bitmap



## 事物	

redis事务的本质，一组命令的集合。一个事务中所有的命令都会被序列化，在事务执行过程中会按照顺序依次执行。相当于把所有命令先放入队列，发起执行命令（exec）后依次执行。

> redis 中的事务没有隔离级别的概念
>
> redis单个命令保持原子性，事务不保证原子性

* 开启事务	multi
* 命令入队    ......
* 执行事务    exec

>  正常执行

```
127.0.0.1:6379> multi		# 开启事务
OK
127.0.0.1:6379> set k1 v1	# 入队
QUEUED
127.0.0.1:6379> set k2 v2	# 入队
QUEUED
127.0.0.1:6379> get k2		# 入队
QUEUED
127.0.0.1:6379> set k3 v3	# 入队
QUEUED
127.0.0.1:6379> exec		# 执行事务
1) OK
2) OK
3) "v2"
4) OK
```

> 放弃事务

```
127.0.0.1:6379> multi		# 开启事务
OK
127.0.0.1:6379> set k1 v1	# 入队
QUEUED
127.0.0.1:6379> set k2 v2	# 入队
QUEUED
127.0.0.1:6379> discard		# 取消事务
```



> 编译型异常（命令有错，代码有问题），事务中所有的命令都不会被执行

```
127.0.0.1:6379> multi		# 开启事务
OK
127.0.0.1:6379> set k1 v1	# 入队
QUEUED
127.0.0.1:6379> set k2 v2	# 入队
QUEUED
127.0.0.1:6379> getset k3	# 错误的命令
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> exec		# 执行事务报错
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k4		# 所有的命令都不会被执行
(nil)
```



> 运行时异常（1/0），其他命令正常执行，错误命令抛出异常

```
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi			# 开启事务
OK	
127.0.0.1:6379> incr k1			# 执行的时候会失败，String不能自增	
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec			# 执行事务，虽然第一个报错，但是其他依旧执行
1) (error) ERR value is not an integer or out of range
2) OK
3) "v2"
```



> 监控，watch

* 悲观锁：认为不管什么情况都会出问题，所以无论做什么都会加锁
* 乐观锁：认为不会出问题，去更新的时候才去判断，如果发生改变，重新拉取，比较并交换

**正常执行**

```
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> incrby out 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20
```

**测试多线程修改值，使用watch当做redis乐观锁操作**

```
127.0.0.1:6379> watch money 		# 监视money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 10
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> exec				# 执行之前，另外一个线程修改了我们的值，这个时候会导致事务失败
1) (integer) 70
2) (integer) 30
```

如果事务失败，重新获取最新的值即可

![image-20200611022710778](C:\Users\luliang\Desktop\Blog\Redis\img\watch)

## Redis.conf详解

