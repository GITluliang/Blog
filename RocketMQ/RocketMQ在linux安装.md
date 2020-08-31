## RocketMQ快速搭建

> 官方文档：http://rocketmq.apache.org/docs/quick-start/

### 1、环境准备

1. 建议使用64位操作系统，建议使用Linux / Unix / Mac；（Windows用户请参见下面的指南）
2. 64位JDK 1.8+;
3. Maven 3.2.x;
4. Git;
5. 适用于Broker服务器的4g +可用磁盘

### 2、安装

```
1、下载安装
cd /srv/ftp
wget https://archive.apache.org/dist/rocketmq/4.7.0/rocketmq-all-4.7.0-bin-release.zip
unzip rocketmq-all-4.7.0-bin-release.zip
mv /srv/ftp/rocketmq-all-4.7.0-bin-release /usr/local/rocketmq
cd /usr/local/rocketmq

2、启动Name Server
nohup sh bin/mqnamesrv &
tail -f ~/logs/rocketmqlogs/namesrv.log
2020-06-28 18:38:05 INFO main - The Name Server boot success. serializeType=JSON

3、启动Broker
nohup sh bin/mqbroker -n 39.107.117.72:9876 &
nohup sh bin/mqbroker -n 39.107.117.72:9876 -c bin/broker.p &
nohup sh bin/mqbroker -n localhost:9876 -c broker.properties autoCreateTopicEnable=true &


tail -f ~/logs/rocketmqlogs/broker.log
2020-06-28 18:39:30 INFO main - The broker[iZ2ze8gkwfpqlsfl8j4o14Z, 172.17.1.189:10911] boot success. serializeType=JSON and name server is localhost:9876

4、如果 tail -f ~/logs/rocketmqlogs/broker.log 提示找不到文件，则打开 当前目录(apache-rocketmq)下的 nohup.out 日志文件查看，发现启动 Broker 失败：无法分配内存

vim bin/runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn512m"

5、官方模拟发消息
export NAMESRV_ADDR=localhost:9876
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer

6、官方模拟收消息
export NAMESRV_ADDR=localhost:9876
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 
 7、关闭服务器
 > sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK

8、常用命令
除了上面几个命令之外，还有如下一些较常用的命令，ip请以实际为准：

查看集群情况： ./mqadmin clusterList -n 127.0.0.1:9876
查看 broker 状态： ./mqadmin brokerStatus -n 127.0.0.1:9876 -b 172.20.1.138:10911
查看 topic 列表： ./mqadmin topicList -n 127.0.0.1:9876
查看 topic 状态： ./mqadmin topicStatus -n 127.0.0.1:9876 -t MyTopic (换成想查询的 topic)
查看 topic 路由： ./mqadmin topicRoute -n 127.0.0.1:9876 -t MyTopic
```

### 3. 注意

* 如果启动Broker失败，修改内存，默认为8g

```
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
```

* 模拟收发消息，容易出现的错误：重复启动，端口被占用

```
https://blog.csdn.net/wenxi2367/article/details/104275594/
```

* 查看Java进程

```
> jps
> ps -ef | grep mq
984 BrokerStartup
953 NamesrvStartup
1567 Jps
```

### 4、控制台安装

> 下载地址：

```
> wget https://github.com/apache/rocketmq-externals/archive/rocketmq-console-1.0.0.tar.gz
> tar -xzvf rocketmq-console-1.0.0.tar.gz -C /usr/local/
> mv /usr/local/rocketmq-externals-rocketmq-console-1.0.0/ /usr/local/rocketmq-console

cd /usr/local/rocketmq-console/

mvn clean install package -Dmaven.test.skip=true

java -jar rocketmq-console-ng-1.0.0.jar

nohup java -jar \
    -Drocketmq.config.namesrvAddr=localhost:9876 \
    -Drocketmq.config.isVIPChannel=false \
    rocketmq-console-ng-1.0.0.jar &

netstat -aptn
```

* 注意

```
注意：如果不对下载的代码做修改的话， 需要设置一个环境变量， 让控制台链接到namesrv。 

设置namesrv环境变量：export NAMESRV_ADDR="localhost:9876"

启动控制台：java -jar rocketmq-console-ng-1.0.0.jar

访问页面：localhost:8080

或者在项目的application.properties中配置namesrv地址
```

