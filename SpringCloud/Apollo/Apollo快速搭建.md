> 官方文档：https://github.com/ctripcorp/apollo/wiki/Quick-Start

## 1、环境准备

* JDK 1.8 +

```
java -version
```

* Mysql 5.6.5+

```
mysql -h127.0.0.1 -uroot -p123456
SHOW VARIABLES WHERE Variable_name = 'version';
```

## 2、数据库配置

### 2.1 创建数据库

Apollo服务端共需要两个数据库：`ApolloPortalDB`和`ApolloConfigDB`，我们把数据库、表的创建和样例数据都分别准备了sql文件，只需要导入数据库即可。

> 注意：如果你本地已经创建过Apollo数据库，请注意备份数据。我们准备的sql文件会清空Apollo相关的表。

### 2.1.1 创建ApolloPortalDB

sql脚本地址：https://github.com/nobodyiam/apollo-build-scripts/blob/master/sql/apolloportaldb.sql

直接导入，然后通过命令进行查看：

```
select `NamespaceId`, `Key`, `Value`, `Comment` from ApolloConfigDB.Item;
```

### 2.1.2 创建ApolloConfigDB

sql脚本地址：https://github.com/nobodyiam/apollo-build-scripts/blob/master/sql/apolloconfigdb.sql

直接导入，然后通过命令进行查看：

```
select `NamespaceId`, `Key`, `Value`, `Comment` from ApolloConfigDB.Item;

```

## 3、安装

```
1、创建目录
mkdir /srv/ftp/apollo/

2、安装包克隆
git clone https://github.com/nobodyiam/apollo-build-scripts.git

3、解压
unzip apollo-quick-start-1.6.1.zip

4、配置数据源
vim demo.sh

#apollo config db info
apollo_config_db_url=jdbc:mysql://localhost:3306/ApolloConfigDB?characterEncoding=utf8
apollo_config_db_username=用户名
apollo_config_db_password=密码（如果没有密码，留空即可）

# apollo portal db info
apollo_portal_db_url=jdbc:mysql://localhost:3306/ApolloPortalDB?characterEncoding=utf8
apollo_portal_db_username=用户名
apollo_portal_db_password=密码（如果没有密码，留空即可）

5、启动执行脚本
./demo.sh start

6、浏览器访问
http://localhost:8070	用户名apollo，密码admin

7、运行客户端程序
./demo.sh client
输入timeout，会看到如下信息：
> timeout
> [SimpleApolloConfigDemo] Loading key : timeout with value: 100

8、客户端查看修改后的值
点击SampleApp进入配置界面，在配置界面timeout修改，然后发布
如果客户端一直在运行的话，在配置发布后就会监听到配置变化，并输出修改的配置信息：

[SimpleApolloConfigDemo] Changes for namespace application
[SimpleApolloConfigDemo] Change - key: timeout, oldValue: 100, newValue: 200, changeType: MODIFIED
再次输入timeout查看对应的值，会看到如下信息：

> timeout
> [SimpleApolloConfigDemo] Loading key : timeout with value: 200
```

## 4. 配置

```
# meta server url
config_server_url=http://localhost:8080
admin_server_url=http://localhost:8090
eureka_service_url=$config_server_url/eureka/
portal_url=http://localhost:8070
```

* localhost:8090

![image-20200627102230102](img\image-20200627101911881.png)

* localhost:8080

![image-20200627102349909](img\image-20200627102349909.png)

* localhost:8070

![image-20200627102153912](img\image-20200627102153912.png)