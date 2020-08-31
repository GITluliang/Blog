## Swagger简介

***前后端分离***

- 前端 -> 前端控制层、视图层
- 后端 -> 后端控制层、服务层、数据访问层
- 前后端通过API进行交互
- 前后端相对独立且松耦合

***产生的问题***

- 前后端集成，前端或者后端无法做到“及时协商，尽早解决”，最终导致问题集中爆发

***解决方案***

- 首先定义schema [ 计划的提纲 ]，并实时跟踪最新的API，降低集成风险

***Swagger***

- 号称世界上最流行的API框架
- Restful Api 文档在线自动生成器 => ***\*API 文档 与API 定义同步更新\****
- 直接运行，在线测试API
- 支持多种语言 （如：Java，PHP等）
- 官网：https://swagger.io/

![image-20200709145117170](img\sw)

## SpringBoot集成Swagger



### 1、导入依赖

```
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

### 2、Swagger配置

```
package com.example.springbootswagger.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.core.env.Profiles;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

@EnableSwagger2 //开启Swagger注解
@Configuration   //配置类
public class SwaggerConfig {
    /**
     * 配置docket以配置Swagger具体参数
     * @param environment
     * @return
     */
    @Bean
    public Docket docket(Environment environment) {
        // 设置要显示swagger的环境，判断当前是否处于该环境
        boolean enable = environment.acceptsProfiles(Profiles.of("dev", "test"));

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .groupName("hello")
                .enable(enable) //配置是否启用Swagger，如果是false，在浏览器将无法访问
                .select()// 通过.select()方法，去配置扫描接口,RequestHandlerSelectors配置如何扫描接口
                .apis(RequestHandlerSelectors.basePackage("com.example.springbootswagger.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    //配置文档信息
    private ApiInfo apiInfo() {
        Contact contact = new Contact("联系人名字", "http://xxx.xxx.com/联系人访问链接", "联系人邮箱");
        return new ApiInfo(
                "Swagger", // 标题
                "学习演示如何配置Swagger", // 描述
                "v1.0", // 版本
                "http://terms.service.url/组织链接", // 组织链接
                contact, // 联系人信息
                "Apach 2.0 许可", // 许可
                "许可链接", // 许可连接
                new ArrayList<>()// 扩展
        );
    }
}
```

测试：http://localhost:8080/swagger-ui.html



### 3、注意点

1、配置多个分组只需要配置多个docket即可：

```
@Bean
public Docket docket1(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("group1");
}
@Bean
public Docket docket2(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("group2");
}
```

2、常用注解

| Swagger注解                                            | 简单说明                                             |
| :----------------------------------------------------- | :--------------------------------------------------- |
| @Api(tags = "xxx模块说明")                             | 作用在模块类上                                       |
| @ApiOperation("xxx接口说明")                           | 作用在接口方法上                                     |
| @ApiModel("xxxPOJO说明")                               | 作用在模型类上：如VO、BO                             |
| @ApiModelProperty(value = "xxx属性说明",hidden = true) | 作用在类方法和属性上，hidden设置为true可以隐藏该属性 |
| @ApiParam("xxx参数说明")                               | 作用在参数、方法和字段上，类似@ApiModelProperty      |

### 4、其他皮肤

1、默认的  访问http://localhost:8080/swagger-ui.html

```
<dependency> 
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

2、bootstrap-ui  访问  http://localhost:8080/doc.html

```
<!-- 引入swagger-bootstrap-ui包 /doc.html-->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>swagger-bootstrap-ui</artifactId>
    <version>1.9.1</version>
</dependency>
```

3、Layui-ui  访问	http://localhost:8080/docs.html

```
<!-- 引入swagger-ui-layer包 /docs.html-->
<dependency>
    <groupId>com.github.caspar-chen</groupId>
    <artifactId>swagger-ui-layer</artifactId>
    <version>1.1.3</version>
</dependency>
```

4、mg-ui  访问	http://localhost:8080/document.html

```
<!-- 引入swagger-ui-layer包 /document.html-->
<dependency>
    <groupId>com.zyplayer</groupId>
    <artifactId>swagger-mg-ui</artifactId>
    <version>1.0.6</version>
</dependency>
```

