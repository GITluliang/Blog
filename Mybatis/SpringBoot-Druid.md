# SpringBoot配置Druid数据源

* 启动 SpringBoot 项目访问地址 http://localhost:8080/druid/login.html 进行查看登录



## 一、纯YML配置：

```java
spring:
  # 数据源配置
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource　　 # MYSQL 5 驱动：com.mysql.jdbc.Driver，MYSQL 6+ 驱动：com.mysql.cj.jdbc.Driver
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/test?autoReconnect=true&useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=CONVERT_TO_NULL&useSSL=false&serverTimezone=GMT%2B8
    username: root
    password: 123456
    # 连接池配置
    druid:
      # 初始化大小，最小，最大
      initial-size: 5
      min-idle: 5
      max-active: 20
      # 配置获取连接等待超时的时间
      max-wait: 60000
      # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位毫秒
      time-between-eviction-runs-millis: 60000
      # 配置一个连接在池中最小生存时间
      min-evictable-idle-time-millis: 300000
      validation-query: SELECT 1 FROM sys_user
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      # 打开 PSCache，并且指定每个连接上 PSCache 的大小
      pool-prepared-statements: true
      max-pool-prepared-statement-per-connection-size: 20
      # 配置监控统计拦截的 Filter，去掉后监控界面 SQL 无法统计，wall 用于防火墙
      filters: stat,wall,log4j
      # 通过 connection-properties 属性打开 mergeSql 功能；慢 SQL 记录
      connection-properties: druid.stat.mergeSql\=true;druid.stat.slowSqlMillis\=5000
      # 配置 DruidStatFilter
      web-stat-filter:
        enabled: true
        url-pattern: /*
        exclusions: .js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*
      # 配置 DruidStatViewServlet
      stat-view-servlet:
        url-pattern: /druid/*
        # IP 白名单，没有配置或者为空，则允许所有访问
        allow: 127.0.0.1
        # IP 黑名单，若白名单也存在，则优先使用
        deny: 192.168.31.253
        # 禁用 HTML 中 Reset All 按钮
        reset-enable: false
        # 登录用户名/密码
        login-username: root
        login-password: 123
```

## 

## 二、yml + Bean配置：

### 1、引POM

```
        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
        <!--druid数据源-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.22</version>
        </dependency>
        <!-- log4j -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
```

### 2. 改YML

application.yml

```java
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://luliang888.top:3306/mybatis_plus?useSSL=false&useUnicode=true&amp;characterEncoding=utf-8
    username: root
    password: 123456
    #使用druid数据源
    type: com.alibaba.druid.pool.DruidDataSource
    #   数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

### 3、配置

config/DruidConfig.java

```java
@Configuration
public class DruidConfig {
    @Bean
    // 将所有前缀为spring.datasource下的配置项都加载DataSource中
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource() {
        return new DruidDataSource();
    }

    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>();
        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "123456");
        initParams.put("allow", "");//默认就是允许所有访问
        initParams.put("deny", "192.168.15.21");
        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        Map<String, String> initParams = new HashMap<>();
        initParams.put("exclusions", "*.js,*.css,/druid/*");
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}
```

## 三、注解：

（1）pom.xml 文件中添加 Druid 依赖（同方式二）

（2）配置文件添加信息（同方式二）

（3）配置 WebServlet

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import javax.servlet.annotation.WebInitParam;
import javax.servlet.annotation.WebServlet;

import com.alibaba.druid.support.http.StatViewServlet;
@WebServlet(urlPatterns = "/druid/*", 
    initParams={
            @WebInitParam(name="allow",value="192.168.16.110,127.0.0.1"),// IP白名单 (没有配置或者为空，则允许所有访问)
            @WebInitParam(name="deny",value="192.168.16.111"),// IP黑名单 (存在共同时，deny优先于allow)
            @WebInitParam(name="loginUsername",value="admin"),// 用户名
            @WebInitParam(name="loginPassword",value="admin"),// 密码
            @WebInitParam(name="resetEnable",value="false")// 禁用HTML页面上的“Reset All”功能
    })
public class DruidServlet extends StatViewServlet {

    private static final long serialVersionUID = 1L;

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

（4）配置 WebFilter

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import javax.servlet.annotation.WebFilter;
import javax.servlet.annotation.WebInitParam;

import com.alibaba.druid.support.http.WebStatFilter;

@WebFilter(filterName="druidWebStatFilter",urlPatterns="/*",
initParams={
    @WebInitParam(name="exclusions",value="*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")// 忽略资源
})
public class DruidFilter extends WebStatFilter {

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

（5）配置启动文件 SpringBootApplication 用于扫描我们配置的类

 添加 @ServletComponentScan 注解进行扫描

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@SpringBootApplication
@ServletComponentScan("com.uy.servlet")
public class SpringBootApplication{

    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication.class, args); } }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

（6）启动 SpringBoot 项目访问地址 http://localhost:8080/druid/login.html 进行查看登录