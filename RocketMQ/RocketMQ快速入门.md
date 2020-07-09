# 名称服务器

NameServer是功能齐全的服务器，主要包括两个功能：

- 在代理管理中，**NameServer**接受来自代理群集的注册，并提供心跳机制以检查代理是否处于活动状态。
- 在路由管理中，每个NameServer将保存有关代理群集的完整路由信息以及客户端查询的**队列**信息。

众所周知，RocketMQ客户端（生产者/消费者）将从NameServer查询队列路由信息，但是客户端如何找到NameServer地址？

有四种方法可以将NameServer地址列表提供给客户端：

- 编程方式，例如`producer.setNamesrvAddr("ip:port")`。
- Java选项，使用`rocketmq.namesrv.addr`。
- 环境变量，使用`NAMESRV_ADDR`。
- HTTP端点。

有关如何查找NameServer地址的更多详细信息，请参阅[此处](http://rocketmq.apache.org/rocketmq/four-methods-to-feed-name-server-address-list/)。

# 经纪服务器

Broker服务器负责消息的存储和传递，消息查询，HA保证等。

如下图所示，Broker服务器具有几个重要的子模块：

- 远程处理模块（代理的条目）处理来自客户端的请求。
- 客户经理，管理客户（生产者/消费者），并维护消费者的主题订阅。
- 存储服务，提供简单的API在物理磁盘中存储或查询消息。
- HA服务，在主代理和从代理之间提供数据同步功能。
- 索引服务，通过指定的键为邮件建立索引，并提供快速的邮件查询。