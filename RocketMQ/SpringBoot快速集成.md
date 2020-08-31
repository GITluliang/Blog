# RocketMQ总结整理

https://blog.csdn.net/asdf08442a/article/details/54882769

# SpringBoot集成RocketMQ

### 依赖

```
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>
```



### yml

```
rocketmq:
  name-server: 127.0.0.1:9876
  producer:
    group: my-group
```



### produce

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestSpringRocketMQ {

    @Autowired
    private RocketMQTemplate rocketMQTemplate;

    @Test
    public void send() {
        /**
         * 同步发送 sync
         * 发送消息采用同步模式，这种方式只有在消息完全发送完成之后才返回结果，此方式存在需要同步等待发送结果的时间代价。
         * 这种方式具有内部重试机制，即在主动声明本次消息发送失败之前，内部实现将重试一定次数，默认为2次（DefaultMQProducer＃getRetryTimesWhenSendFailed）。
         * 发送的结果存在同一个消息可能被多次发送给给broker，这里需要应用的开发者自己在消费端处理幂等性问题。
         */
        SendResult syncSend = rocketMQTemplate.syncSend("my-topic", "syncSend");
        System.out.println(syncSend);
        /**
         * 异步发送 async
         * 发送消息采用异步发送模式，消息发送后立刻返回，当消息完全完成发送后，会调用回调函数sendCallback来告知发送者本次发送是成功或者失败。
         * 异步模式通常用于响应时间敏感业务场景，即承受不了同步发送消息时等待返回的耗时代价。同同步发送一样，异步模式也在内部实现了重试机制，
         * 默认次数为2次（DefaultMQProducer#getRetryTimesWhenSendAsyncFailed}）。发送的结果同样存在同一个消息可能被多次发送给给broker，需要应用的开发者自己在消费端处理幂等性问题。
         */
        rocketMQTemplate.asyncSend("my-topic", "syncSend", new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {
                System.out.println("send successful");
            }

            @Override
            public void onException(Throwable throwable) {
                System.out.println("send fail; {}" + throwable.getMessage());
            }
        });

        /**
         * 直接发送 one-way
         * 采用one-way发送模式发送消息的时候，发送端发送完消息后会立即返回，不会等待来自broker的ack来告知本次消息发送是否完全完成发送。
         * 这种方式吞吐量很大，但是存在消息丢失的风险，所以其适用于不重要的消息发送，比如日志收集
         */
        rocketMQTemplate.sendOneWay("my-topic", "sendOneWay");

        rocketMQTemplate.convertAndSend("my-topic", "convertAndSend");
    }
}
```

### consume

```
@Component
@RocketMQMessageListener(
        topic = "my-topic",
        consumerGroup = "my-group",
        selectorExpression = "*"
)
public class SpringConsumer implements RocketMQListener<String> {

    @Override
    public void onMessage(String msg) {
        System.out.println("接收到消息 -> " + msg);
    }
}
```

## 