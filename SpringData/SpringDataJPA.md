## @整合SpringData JPA

### 1）、SpringData简介

![](img\搜狗截图20180306105412.png)

### 2）、整合SpringData JPA

#### pom

```java
        <!--data-jpa-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>            
```

#### yml

```java
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://luliang888.top:3306/mybatis_plus?useSSL=false&useUnicode=true&amp;characterEncoding=utf-8
    username: root
    password: 123456
  jpa:
    hibernate:
      #     更新或者创建数据表结构
      ddl-auto: update
    #    控制台显示SQL
    show-sql: true

```

#### 主启动

```java
@SpringBootApplication
public class SpringdatajpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringdatajpaApplication.class, args);
    }

}
```

#### 业务类

entity

```java
//使用JPA注解配置映射关系
@Entity //告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "user") //@Table来指定和哪个数据表对应;如果省略默认表名就是user；
@Data
@Accessors(chain = true)
public class User {
    @Id //这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增主键
    private Long id;
    @Column //省略默认列名就是属性名
    private String name;
    @Column(name = "age",length = 50) //这是和数据表对应的一个列
    private Integer age;
    private String email;
    private String nickName;
    private Date createTime;
    private Date updateTime;
    private Integer version;
    private Integer deleted;
}
```

repository接口

```
//继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}
```

Test

```java
@SpringBootTest
@RunWith(SpringRunner.class)
class SpringdatajpaApplicationTests {

    @Resource
    private UserRepository userRepository;

    @Test
    void contextLoads() {
        List<User> all = userRepository.findAll();
        all.forEach(System.out::println);
    }
}
```

