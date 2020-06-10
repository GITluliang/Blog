### @SpringBootApplication：

*  来标注一个主程序类，说明这是一个Spring Boot应用

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

### @SpringBootConfiguration/@Configuration/@Bean

* SpringBootConfiguration：SpringBoot定义的注解配置类

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
```

* @Configuration：Spring定义的注解配置类

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
```

* @Bean：用在方法上，表示将方法产生的结果结果交给Spring容器进行管理；

###  @MapperScan/@Mapper

* mybatis提供的mapper接口注解，对mapper实现动态代理，然后在交由Spring容器进行管理。

* @Mapper，最终 Mybatis 会有一个拦截器，会自动的把 @Mapper 注解的接口生成动态代理类。这点可以在 MapperRegistry 类中的源代码中查看。@Mapper 注解针对的是一个一个的类，相当于是一个一个 Mapper.xml 文件。而一个接口一个接口的使用 @Mapper，太麻烦了，于是 @MapperScan 就应用而生了。
* @MapperScan 配置一个或多个包路径，自动的扫描这些包路径下的类，自动的为它们生成代理类
* 当使用了 @MapperScan 注解，将会生成 MapperFactoryBean， 如果没有标注 @MapperScan 也就是没有 MapperFactoryBean 的实例，就走 @Import 里面的配置，具体可以在 AutoConfiguredMapperScannerRegistrar 和 MybatisAutoConfiguration 类中查看源代码进行分析。

### @Resource/@Autowired

* @Resource和@Autowired都是做bean的注入时使用

* @Autowired：为Spring提供注解
* @Resource：为J2EE提供的注解，推荐使用

### @Component/@Repository/@Service/@Controller

* 都可以依赖可以实现bean注入，@Repository一般用于mapper层，@Service一般用于service层，@Controller一般用于控制层，前面三个都包含@Component，把普通类注入到spring容器中。

### @ComponentScan

* Spring提供的包扫描注解，主要就是定义**扫描的路径**从中找出标识了**需要装配**的类自动装配到spring的bean容器中
* SpringBoot默认是从主启动类下开始扫描
* Spring项目中一定要使用Spring包扫描，定义在配置类上

* **示例：Spring项目中使用注解方式进行容器启动**
  * 配置类：

    @Configuration
    //添加自动扫描注解，basePackages为TestBean包路径
    @ComponentScan(basePackages = "com.dxz.demo.configuration")
    public class TestConfiguration {}

* ​	主启动：

    public static void main(String[] args) {
        // @Configuration注解的spring容器加载方式，用AnnotationConfigApplicationContext替换ClassPathXmlApplicationContext
        ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);
     
        // 如果加载spring-context.xml文件：
        // ApplicationContext context = new
        // ClassPathXmlApplicationContext("spring-context.xml");
        
         //获取bean
        TestBean tb = (TestBean) context.getBean("testBean");
        tb.sayHello();

### @ConfigurationProperties/@Value

* 获取配置文件yml或properties中的值；

* 如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

* 如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；

### @PropertySource&@ImportResource&

* @**PropertySource**：加载指定的配置文件；

```
@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
public class Person {}
```

* @**ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效；

  Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

  想让Spring的配置文件生效，加载进来；@**ImportResource**标注在一个配置类上

```java
@ImportResource(locations = {"classpath:beans.xml"})
导入Spring的配置文件让其生效
```

* SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

  ​	1、配置类**@Configuration**------>Spring配置文件

  ​	2、使用**@Bean**给容器中添加组件