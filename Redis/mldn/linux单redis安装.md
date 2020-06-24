## reids单实例安装
```
1、更新依赖
apt-get update
apt-get -y install make g++ gcc libpcre3 libpcrecpp* libpcre3-dev libssl-dev autoconf automake libtool libncurses5-dev libaio.dev ruby-dev rubygems

2、上传或下载安装包
cd /srv/ftp
wget http://download.redis.io/releases/redis-5.0.5.tar.gz

3、解压
tar xzvf /srv/ftp/redis-5.0.5.tar.gz -C /usr/local/src/

4、重命名
mv /usr/local/src/redis-5.0.5/ /usr/local/src/redis

5、进行安装目录
cd /usr/local/src/redis/

6、编译，安装

make	

make install	# 正常安装,redis的默认安装路径/usr/local/bin		
make install PREFIX=/usr/local/bin/redis-6.0.5	#如果以前安装过，指定安装路径

7、将所有内存交由应用程序管理
echo "vm.overcommit_memory=1" >> /etc/sysctl.conf

8、将配置写入linux内核中
/sbin/sysctl -p
```

## redis配置

```
1、创建redis目录
mkdir -p /usr/local/redis/{bin,conf}

2、拷贝启动文件、连接文件、测试文件
cp /usr/local/src/redis/src/redis-server /usr/local/redis/bin/
cp /usr/local/src/redis/src/redis-cli /usr/local/redis/bin/
cp /usr/local/src/redis/src/redis-benchmark /usr/local/redis/bin/

3、配置文件
cp /usr/local/src/redis/redis.conf /usr/local/redis/conf/

4、数据存储目录
	# run启动进程编号	logs日志	dbcache缓存数据
mkdir -p /usr/data/redis/{run,logs,dbcache}

5、修改配置文件
vim /usr/local/redis/conf/redis.conf

6、修改内容
port 6379		# 端口
daemonize yes	# 后台启动
pidfile /usr/data/redis/run/redis_6379.pid		# 进程目录
logfile "/usr/data/redis/logs/redis_6379.log"	# 日志目录
databases 16	# 数据库实例								
dir /usr/data/redis/dbcache	# 缓存数据目录

7、指定配置文件启动
/usr/local/redis/bin/redis-server /usr/local/redis/conf/redis.conf 

8、查看端口
netstat -nptl
ps -ef|grep redis

9、进行客户端连接
/usr/local/redis/bin/redis-cli

10、通过命令进行测试
set lee hello
get lee

11、进行性能测试
/usr/local/redis/bin/redis-benchmark -n 10000 -d 50 -c 2000
```

## redis安全配置

```
1、修改配置文件
vim /usr/local/redis/conf/redis.conf 

# bind 127.0.0.1	# 注释掉，或者填写本机ip
requirepass 123456	# 连接密码

2、重新启动
killall redis-server
/usr/local/redis/bin/redis-server /usr/local/redis/conf/redis.conf 

3、进行连接
/usr/local/redis/bin/redis-cli -h iZ2ze8gkwfpqlsfl8j4o14Z -p 6379
auth 123456
```

## redis-stat性能监控

```
1、创建存放目录
mkdir /usr/local/redis/tools
cd /usr/local/redis/tools

2、下载
cd /usr/local/src/
git clone https://github.com/junegunn/redis-stat

3、安装ruby依赖
apt-get -y install ruby ruby-dev rubygems

4、进行安装
cd /usr/local/redis/tools/redis-stat/bin
gem install redis-stat

5、启动监控
/usr/local/redis/tools/redis-stat/bin/redis-stat 127.0.0.1:6379 -a 123456

8、启动性能测试进行观察
/usr/local/redis/bin/redis-benchmark -n 100000 -d 500 -c 1000

9、通过web进行监控
/usr/local/redis/tools/redis-stat/bin/redis-stat 127.0.0.1:6379 -a 123456 --server=8080 --daemon --verbose

10、通过浏览器进行访问
127.0.0.1:8080
```

