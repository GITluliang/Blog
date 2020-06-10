## SpringBoot异步任务

在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的；但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，其实，在Spring 3.x之后，就已经内置了@Async来完美解决这个问题。

**两个注解**：@EnableAysnc、@Aysnc

* @EnableAysnc

```java
@EnableAsync  //开启异步注解功能
@SpringBootApplication
public class Springboot04TaskApplication {
	public static void main(String[] args) {
		SpringApplication.run(Springboot04TaskApplication.class, args);
	}
}
```

* @Aysnc

```java
@Service
public class AsyncService {
    //告诉Spring这是一个异步方法
    @Async
    public void hello(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("处理数据中...");
    }
}
```

------



## SpringBoot定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。Spring为我们提供了异步执行任务调度的方式，提供TaskExecutor 、TaskScheduler 接口。

​	**两个注解：**@EnableScheduling、@Scheduled

* @EnableScheduling

```java
@EnableScheduling //开启基于注解的定时任务
@SpringBootApplication
public class Springboot04TaskApplication {
   public static void main(String[] args) {
      SpringApplication.run(Springboot04TaskApplication.class, args);
   }
}
```

* @Scheduled

```java
@Service
public class ScheduledService {

    /**
     * second(秒), minute（分）, hour（时）, day of month（日）, month（月）, day of week（周几）.
     * 0 * * * * MON-FRI
     *  【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次
     *  【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次
     *  【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次
     *  【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次
     *  【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；
     */
   // @Scheduled(cron = "0 * * * * MON-SAT")
    //@Scheduled(cron = "0,1,2,3,4 * * * * MON-SAT")
   // @Scheduled(cron = "0-4 * * * * MON-SAT")
    @Scheduled(cron = "0/4 * * * * MON-SAT")  //每4秒执行一次
    public void hello(){
        System.out.println("hello ... ");
    }
}
```

**cron表达式：**

![image-20200525092053275](images\image-20200525092014925.png)

------



## SpringBoot邮件任务

官方地址：

```
https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#mail

```

![image-20200525092206166](C:\Users\luliang\Desktop\Blog\SpringBoot\images\image-20200525092206166.png)

* 1、邮件发送需要引入spring-boot-starter-mail

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

* 2、Spring Boot 自动配置MailSenderAutoConfiguration



* 定义MailProperties内容，配置在application.yml中

```
spring.mail.username=534096094@qq.com
spring.mail.password=gtstkoszjelabijb
spring.mail.host=smtp.qq.com
spring.mail.properties.mail.smtp.ssl.enable=true
```

​	自动装配JavaMailSender：JavaMailSenderImpl

* 测试邮件发送

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot04TaskApplicationTests {

   @Autowired
   JavaMailSenderImpl mailSender;

   @Test
   public void contextLoads() {
      SimpleMailMessage message = new SimpleMailMessage();
      //邮件设置
      message.setSubject("通知-今晚开会");
      message.setText("今晚7:30开会");

      message.setTo("17512080612@163.com");
      message.setFrom("534096094@qq.com");

      mailSender.send(message);
   }

   @Test
   public void test02() throws  Exception{
      //1、创建一个复杂的消息邮件
      MimeMessage mimeMessage = mailSender.createMimeMessage();
      MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);

      //邮件设置
      helper.setSubject("通知-今晚开会");
      helper.setText("<b style='color:red'>今天 7:30 开会</b>",true);

      helper.setTo("17512080612@163.com");
      helper.setFrom("534096094@qq.com");

      //上传文件
      helper.addAttachment("1.jpg",new File("C:\\Users\\lfy\\Pictures\\Saved Pictures\\1.jpg"));
      helper.addAttachment("2.jpg",new File("C:\\Users\\lfy\\Pictures\\Saved Pictures\\2.jpg"));

      mailSender.send(mimeMessage);

   }

}
```

