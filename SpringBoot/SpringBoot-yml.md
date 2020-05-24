* application.properties等同于application.yml



### YML文件常用配置

* 日志：

```
logging:
  level:
    root: info
    com.nuoze.cctower: debug
    file: ./data/log/cctower.log
```

* Spring数据源：

```
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://luliang888.top:3306/mybatis_plus?useSSL=false&useUnicode=true&amp;characterEncoding=utf-8
    username: root
    password: 123456
```

