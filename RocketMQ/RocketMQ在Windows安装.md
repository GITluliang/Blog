### RocketMQ下载地址及相关文档

下载地址：http://rocketmq.apache.org/dowloading/releases

下载版本 rocketmq-all-4.6.0-bin-release.zip解压后目录结构，bin目录下存放可运行的脚本

RocketMQ自身分为 NameServer 和 Broker 两个部分，因此，用作本机开发调试用的最小应用，应该分别启动一个NameServer和一个Broker节点。

RocketMQ默认提供了 windows环境 和 linux环境 下的启动脚本。脚本位于bin目录下，windows的脚本以.cmd为文件名后缀，linux环境的脚本以.sh为文件名后缀。不过，通常情况下，windows下的脚本双击启动时，都是窗口一闪而过，启动失败。下面的内容就帮大家解决这些问题

#### 第一步，配置 JAVA_HOME 和 ROCKETMQ_HOME 环境变量

> JAVA_HOME 的配置这里不再赘述
> ROCKETMQ_HOME 应配置为D:\programs\rocketmq\rocketmq-all-4.4.0-bin-release

#### 第二步，启动 NameServer

> NameServer的启动脚本是bin目录下的mqnamesrv.cmd。
> 上文讲过，即使配置好了ROCKETMQ_HOME环境变量，mqnamesrv.cmd的启动通常也以失败告终。
> 阅读mqnamesrv.cmd脚本，发现其实际上是调用了runserver.cmd脚本来实现启动的动作。
> 而在runserver.cmd脚本，java的默认启动参数中，启动时堆内存的大小为2g，老旧一点的机器上根本没有这么多空闲内存。

因此，用编辑器修改一下runserver.cmd脚本。将原来的内存参数注释掉（cmd脚本使用rem关键字），修改为：

```
rem set "JAVA_OPT=%JAVA_OPT% -server -Xms2g -Xmx2g -Xmn1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
    set "JAVA_OPT=%JAVA_OPT% -server -Xms256m -Xmx512m"
```

直接双击mqnamesrv.cmd脚本启动NameServer。

或cmd窗口执行 

```
> mqnamesrv.cmd -n localhost:9876
```



看到 The Name Server boot success 字样，表示NameServer己启动成功。

windows环境下，可以在目录%USERPROFILE%\logs\rocketmqlogs下找到NameServer的启动日志。文件名为namesrv.log。

#### 第三步，启动 Broker

Broker的启动脚本是mqbroker.cmd。

与mqnamesrv.cmd脚本类似，mqbroker.cmd是调用runbroker.cmd脚本启动Broker的。

同样的，优化一下runbroker.cmd的启动内存

```
rem set "JAVA_OPT=%JAVA_OPT% -server -Xms2g -Xmx2g -Xmn1g"
    set "JAVA_OPT=%JAVA_OPT% -server -Xms256m -Xmx512m"
```

此外，Broker脚本启动之前要指定 NameServer的地址。

NameServer默认启动端口是9876，这点可以从NameServer的启动日志中找到记录。

因此，还需要修改mqbroker.cmd脚本，增加NameServer的地址。

```
> mqbroker.cmd -n localhost:9876 autoCreateTopicEnable=true
```

mqbroker.cmd -n localhost:9876 autoCreateTopicEnable=true

双击mqbroker.cmd脚本启动Broker。



看到 The broker ... boot success 字样，表示Broker己启动成功。

#### 四、安装可视化插件 

\1. 下载地址：https://github.com/apache/rocketmq-externals.git下载完成之后，进入‘rocketmq-externals\rocketmq-console\src\main\resources’文件夹，打开‘application.properties’进行配置。

```
server.contextPath=
server.port=8088    #此处配置工程端口
#spring.application.index=true
spring.application.name=rocketmq-console
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
logging.config=classpath:logback.xml
#if this value is empty,use env value rocketmq.config.namesrvAddr  NAMESRV_ADDR | now, you can set it in ops page.default localhost:9876
rocketmq.config.namesrvAddr=127.0.0.1:9876   #此处配置MQ地址
#if you use rocketmq version < 3.5.8, rocketmq.config.isVIPChannel should be false.default true
rocketmq.config.isVIPChannel=
#rocketmq-console's data path:dashboard/monitor
rocketmq.config.dataPath=/tmp/rocketmq-console/data
#set it false if you don't want use dashboard.default true
rocketmq.config.enableDashBoardCollect=true
```

#### 2. 编译启动

用CMD进入‘\rocketmq-externals\rocketmq-console’文件夹，执行‘mvn clean package -Dmaven.test.skip=true’，编译生成。

编译成功之后，Cmd进入‘target’文件夹，执行‘java -jar rocketmq-console-ng-1.0.0.jar’，启动‘rocketmq-console-ng-1.0.0.jar’。

**3.测试**

浏览器中输入‘127.0.0.1:8088’，成功后即可查看。



#### 4消息清除

清空文件夹内容：C:\Users\Administrator\store

​              C:\Users\Administrator\logs