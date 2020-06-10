# ElasticSearch

## 一、安装与配置

Elasticsearch 7.2.1 目录结构如下：

- bin ：脚本文件，包括 ES 启动 & 安装插件等等
- config ： elasticsearch.yml（ES 配置文件）、jvm.options（JVM 配置文件）、日志配置文件等等
- JDK ： 内置的 JDK，JAVA_VERSION="12.0.1"
- lib ： 类库
- logs ： 日志文件
- modules ： ES 所有模块，包括 X-pack 等
- plugins ： ES 已经安装的插件。默认没有插件
- data ： ES 启动的时候，会有该目录，用来存储文档数据。该目录可以设置

1. 下载地址：

```
https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.7.0-linux-x86_64.tar.gz
```

ES下载缓慢，百度云地址参考：https://blog.csdn.net/weixin_37281289/article/details/101483434

2. 安装与配置：

```
tar xzvf elasticsearch-7.6.0-linux-x86_64.tar.gz -C /usr/local/
cd /usr/local/elasticsearch-7.6.0/
```

* 修改内存大小：

```
mkdir data
vim config/jvm.options	//修改默认jvm内存，默认1g
```

* 修改目录：vim config/elasticsearch.yml

```
path.data: /usr/local/elasticsearch-7.6.0/data
path.logs: /usr/local/elasticsearch-7.6.0/logs
```

### 1、root账号启动报错

```
groupadd elsearch								//创建组
useradd elsearch -g elsearch -p elasticsearch	//创建用户
cd /usr/local/									
chown -R elsearch:elsearch  elasticsearch-7.6.0 //通过chown改变文件的拥有者和群组		
su elsearch										//切换用户
cd /usr/local/elasticsearch-7.6.0/bin			//进入安装目录
./elasticsearch									//启动
./elasticsearch -d								//后台启动

ps -ef | grep Elasticsearch						//查看启动
kill -15 xxx									//终止服务
```

* 打开终端进行测试：

```
curl http://localhost:9200
curl 'http://localhost:9200/?pretty'
```

### 2、Elasticsearch外网访问

1. 打开配置文件，添加如下配置：

```
vim /usr/local/elasticsearch-7.6.0/config/elasticsearch.yml
```

```
network.host: 0.0.0.0
```

2. 启动会报错：

```
ERROR: [2] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[2]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

3. 解决方案：

* 问题1解决：使用sudo用户修改文件：`sudo vim /etc/sysctl.conf`，添加一行配置：

```
vm.max_map_count=655360
```

​	查看是否生效：sudo sysctl -p

* 问题2解决：config/elasticsearch.yml放开如下配置

```
cluster.initial_master_nodes: ["node-1", "node-2"]
```





## 二、ElasticSearch使用

1.索引		==》 数据库



2.类型		==》 表



3.文档		==》 表记录



4.属性		==》 字段

