## 1. String数据类型

​	字符串是使用最多的一种数据类型，C语言当中，0代表false，1代表true

>SET key value [EX seconds|PX milliseconds] [NX|XX] [KEEPTTL]	# 设置值
>
>GET key											  # 获取值
>
>SETEX key seconds value				# 设置失效时间
>
>MSET key value [key value ...]		 # 进行多个内容设置
>
>SETNX key value								# 不覆盖单个内容设置
>
>MSETNX key value [key value ...]	# 不覆盖设置多个内容
>
>APPEND key value							# 追加
>
>STRLEN key										# 获取长度
>
>DEL key [key ...]								 # 删除key
>
>flushdb [ASYNC]								# 清空当前库
>
>FLUSHALL [ASYNC]							# 清空所有库
>
>SELECT index									# 切换库

```
root@iZ2ze8gkwfpqlsfl8j4o14Z:~# redis-cli -p 6379
127.0.0.1:6379> AUTH 123456
OK
127.0.0.1:6379> set msg lee-hello
OK
127.0.0.1:6379> get msg
"lee-hello"
127.0.0.1:6379> set info {name:SMITH,age:18}
OK
127.0.0.1:6379> set code 654321 EX 10
OK
127.0.0.1:6379> mset info-a lee info-b hello info-c world
OK
127.0.0.1:6379> setnx info hello
(integer) 0
127.0.0.1:6379> setnx infox hello
(integer) 1
127.0.0.1:6379> msetnx info-a lee info-b hello info-d xiaoxiao
(integer) 0
127.0.0.1:6379> append msg !!!
(integer) 12
127.0.0.1:6379> strlen msg
(integer) 12
127.0.0.1:6379> del msg info
(integer) 2
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> select 1
OK
```

## 2. Hash数据类型

Hash是对String类型的扩展，可以理解为key-map，可以创建多个子key来保存一组数据。

>HSET key field value [field value ...]		# 设置hash值
>
>HGET key field											# 获取hash
>
>HMSET key field value [field value ...]	# 多个hash设置
>
>HMGET key field [field ...]						 # 获取多个子key的值
>
>HSETNX key field value							 # 不覆盖进行设置
>
>HDEL key field [field ...]							 # 删除
>
>HVALS key													# 获取hash key中的所有数据
>
>HGETALL key												# 获取hash中的所有内容，包括key和value
>
>
>
>INCR key												# 数字操作，自增1
>
>INCRBY key increment						# 数字操作，自增指定长度
>
>DECR key												# 数字操作，自减1
>
>DECRBY key decrement					   # 自减指定长度
>
>HINCRBY key field increment			 # 对hash 类型进行自增

```
root@iZ2ze8gkwfpqlsfl8j4o14Z:~# redis-cli 
127.0.0.1:6379> AUTH 123456
OK
127.0.0.1:6379> hset member-lee name xiaoli
(integer) 1
127.0.0.1:6379> hset member-lee age 18
(integer) 1
127.0.0.1:6379> hset member-lee salary 1.3
(integer) 1
127.0.0.1:6379> hmset member-xiaoke name xb age 69 salary -103.2
OK
127.0.0.1:6379> hmget member-lee name age salary
1) "xiaoli"
2) "18"
3) "1.3"
127.0.0.1:6379> hsetnx member-lee name xiaoli
(integer) 0
127.0.0.1:6379> hdel member-lee age salary
(integer) 2
127.0.0.1:6379> hvals member-xiaoke
1) "xb"
2) "69"
3) "-103.2"
127.0.0.1:6379> hgetall member-xiaoke
1) "name"
2) "xb"
3) "age"
4) "69"
5) "salary"
6) "-103.2"
127.0.0.1:6379> set xiaohei-age 69
OK
127.0.0.1:6379> incr xiaohei-age
(integer) 70
127.0.0.1:6379> incrby xiaohei-age 78
(integer) 148
127.0.0.1:6379> decr xiaohei-age
(integer) 147
127.0.0.1:6379> decrby xiaohei-age 200
(integer) -53
127.0.0.1:6379> hset shopcar-lee 99 1
(integer) 1
127.0.0.1:6379> hincrby shopcar-lee 99 1
(integer) 2
```

## 3. List数据类型

List数据类型主要实现的是一个双端队列的处理概念（可以实现队列的入队和出队操作），也可实现队头和队尾的数据操作。

>LPUSH key element [element ...]							# 向队列左边进行数据添加
>
>RPUSH key element [element ...]							# 向队列右边进行数据添加
>
>LRANGE key start stop											  #  获取队列内容，全部获取0 -1
>
>LPOP key																	# 左边弹出
>
>RPOP key																	# 右边弹出
>
>LINSERT key BEFORE|AFTER pivot element			# 在指定位置之前|之后进行添加
>
>LSET key index element								# 根据索引进行内容修改
>
>LREM key count element								# 进行指定个数的删除，有可能存在重复数据
>
>LTRIM key start stop										# 删除当前位置之外的数据
>
>RPOPLPUSH source destination					# 将弹出的内容保存到另外一个集合
>
>LLEN key															# 获取队列长度

```
127.0.0.1:6379> lpush lee-message hello-a hello-b hello-c hello-d hello-e
(integer) 5
127.0.0.1:6379> rpush lee-message world-1 world-2 world-3 world-4 world-5
(integer) 10
127.0.0.1:6379> lrange lee-message 0 -1
 1) "hello-e"
 2) "hello-d"
 3) "hello-c"
 4) "hello-b"
 5) "hello-a"
 6) "world-1"
 7) "world-2"
 8) "world-3"
 9) "world-4"
10) "world-5"
127.0.0.1:6379> lpop lee-message
"hello-e"
127.0.0.1:6379> rpop lee-message
"world-5"
127.0.0.1:6379> linsert lee-message BEFORE hello-c hello-before
(integer) 9
127.0.0.1:6379> lrange lee-message 0 -1
1) "hello-d"
2) "hello-before"
3) "hello-c"
4) "hello-b"
5) "hello-a"
6) "world-1"
7) "world-2"
8) "world-3"
9) "world-4"
127.0.0.1:6379> linsert lee-message AFTER hello-c hello-after
(integer) 10
127.0.0.1:6379> lset lee-message 3 hello-world
OK
127.0.0.1:6379> lpush lee-message hello-repeat hello-repeat hello-repeat
(integer) 13
127.0.0.1:6379> lrem lee-message 1 hello-e
(integer) 0
127.0.0.1:6379> lrem lee-message 2 hello-repeat
(integer) 2
127.0.0.1:6379> ltrim lee-message 2 4
OK
127.0.0.1:6379> rpoplpush lee-message lee-pop
"hello-world"
127.0.0.1:6379> llen lee-pop
(integer) 1
```

## 4. Set数据类型

Set集合最大的特点是可以保存无重复的元素集合，每个集合里面最多可保存40亿元素的信息，set集合出现的最大意义在于可以方便的进行各种集合运算。

>SADD key member [member ...]			# 设置set值
>
>SMEMBERS key										 # 查看内容
>
>SPOP key [count]									# 弹出数据，可以指定个数
>
>SRANDMEMBER key [count]				# 弹出数据内容，不进行元素删除
>
>SDIFF key [key ...]									   # 计算多个集合的差值
>
>SINTER key [key ...]									# 计算多个集合交集
>
>SUNION key [key ...]								  # 计算多个集合并集
>
>SDIFFSTORE destination key [key ...]		# 进行差集信息的存储
>
>SINTERSTORE destination key [key ...]	  # 实现交集信息存储
>
>SUNIONSTORE destination key [key ...]	# 实现并集信息存储
>
>SMOVE source destination member	# 由一个set向另一个set进行元素移动
>
>SISMEMBER key member			# 判断是否存在某个元素

```
127.0.0.1:6379> sadd user-lee a b c d e f g h i h k
(integer) 1
127.0.0.1:6379> sadd user-hei a b c d x y z a a d c
(integer) 0
127.0.0.1:6379> smembers user-lee
 1) "i"
 2) "h"
 3) "e"
 4) "f"
 5) "a"
 6) "g"
 7) "d"
 8) "k"
 9) "c"
10) "b"
127.0.0.1:6379> spop user-lee
"c"
127.0.0.1:6379> srandmember user-lee 
"a"
127.0.0.1:6379> sdiff user-lee user-hei
1) "f"
2) "e"
3) "h"
4) "i"
5) "k"
6) "g"
127.0.0.1:6379> sinter user-lee user-hei
1) "d"
2) "a"
3) "b"
127.0.0.1:6379> sunion user-lee user-hei
 1) "y"
 2) "i"
 3) "f"
 4) "e"
 5) "h"
 6) "z"
 7) "a"
 8) "x"
 9) "g"
10) "d"
11) "k"
12) "c"
13) "b"
127.0.0.1:6379> sdiffstore user-diff user-lee user-hei
(integer) 6
127.0.0.1:6379> sinterstore user-inter user-lee user-hei
(integer) 3
127.0.0.1:6379> sunionstore user-union user-lee user-hei
(integer) 13
127.0.0.1:6379> smove user-lee user-hei k
(integer) 1
127.0.0.1:6379> sismember user-lee a
(integer) 1
```



## 5. ZSet数据类型

ZSet也属于Set集合，是对set集合的扩充，区别在于：第一可以进行顺序排序，第二可以进行数据打分。例如在很多搜索引擎中会出现一些热点的搜索关键字，就可以通过ZSet数据类型来实现。

>ZADD key [NX|XX] [CH] [INCR] score member [score member ...]	# 内容设置
>
>ZRANGE key start stop [WITHSCORES]			# 显示内容
>
>ZREVRANGE key start stop [WITHSCORES]	# 进行排序



```
127.0.0.1:6379> zadd keyword-20201010 5.0 preety
(integer) 1
127.0.0.1:6379> zadd keyword-20201010 2.0 spring
(integer) 1
127.0.0.1:6379> zadd keyword-20201010 10.0 xiaoli
(integer) 1
127.0.0.1:6379> zadd keyword-20201010 3.5 stream
(integer) 1
127.0.0.1:6379> zrange keyword-20201010 0 -1
1) "spring"
2) "stream"
3) "preety"
4) "xiaoli"
127.0.0.1:6379> zrange keyword-20201010 0 -1 WITHSCORES
1) "spring"
2) "2"
3) "stream"
4) "3.5"
5) "preety"
6) "5"
7) "xiaoli"
8) "10"
127.0.0.1:6379> zrevrange keyword-20201010 0 -1
1) "xiaoli"
2) "preety"
3) "stream"
4) "spring"
127.0.0.1:6379> zrevrange keyword-20201010 1 5
1) "preety"
2) "stream"
3) "spring"
```



















