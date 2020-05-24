

## 1、主键策略

**自3.3.0开始,默认使用雪花算法+UUID(不含中划线)**

**（1）ID_WORKER**

MyBatis-Plus默认的主键策略是：ID_WORKER  *全局唯一ID*

**参考资料：分布式系统唯一ID生成方案汇总：**https://www.cnblogs.com/haoxinyue/p/5208136.html

**（2）自增策略**

- 要想主键自增需要配置如下主键策略

- - 需要在创建数据表的时候设置主键自增
  - 实体字段中配置 @TableId(type = IdType.AUTO)

```
@TableId(type = IdType.AUTO)
private Long id;
```

要想影响所有实体的配置，可以设置全局主键配置

```java
#全局设置主键生成策略
mybatis-plus.global-config.db-config.id-type=auto
```

其它主键策略：分析 IdType 源码可知

```
public enum IdType {
    AUTO(0),		//数据库ID自增
    NONE(1),		//该类型为未设置主键类型
    INPUT(2),		//用户输入ID,该类型可以通过自己注册自动填充插件进行填充
    ASSIGN_ID(3),	//支持自动转换为String类型，但数值类型不支持自动转换，需精准匹配，例如返回Long，实体主键就不支持定义为Integer。类型：Long,Integer,String
    ASSIGN_UUID(4),	//默认不含中划线的UUID生成。类型：String
    
    //*****以下过期*****
    @Deprecated
    ID_WORKER(3),		//全局唯一ID (idWorker)
    @Deprecated
    ID_WORKER_STR(3),	//字符串全局唯一ID (idWorker 的字符串表示)
    @Deprecated
    UUID(4);			//全局唯一ID (UUID)
```

​	idWorker、idWorker、UUID，只有当插入对象ID 为空，才自动填充。

## 2、自动填充

项目中经常会遇到一些数据，每次都使用相同的方式填充，例如记录的创建时间，更新时间等。

我们可以使用MyBatis Plus的自动填充功能，完成这些字段的赋值工作：

**（1）数据库表中添加自动填充字段**

在User表中添加datetime类型的新的字段 create_time、update_time

**（2）实体上添加注解**

```java
@Data
@Accessors(chain = true)
public class User {
	
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
}
```

支持类型：

```java
public enum FieldFill {
    DEFAULT,	//删除填充
    INSERT,		//添加填充
    UPDATE,		//更新填充
    INSERT_UPDATE;	//添加或更新时填充

    private FieldFill() {
    }
}
```

​	

（3）实现元对象处理器接口

**注意：不要忘记添加 @Component 注解**

```java
@Component
@Slf4j
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill ....");
        this.setFieldValByName("createTime", new Date(), metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill ....");
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
```

## 3、乐观锁

**主要适用场景：**当要更新一条记录的时候，希望这条记录没有被别人更新，也就是说实现线程安全的数据更新

**乐观锁实现方式：**

- 取出记录时，获取当前version
- 更新时，带上这个version
- 执行更新时， set version = newVersion where version = oldVersion
- 如果version不对，就更新失败

***（1）数据库中添加version字段***

```java
ALTER TABLE `user` ADD COLUMN `version` INT
```

***（2）实体类添加version字段***

并添加 @Version 注解

```java
@Version
@TableField(fill = FieldFill.INSERT)
private Integer version;
```

***（3）元对象处理器接口添加version的insert默认值***

```java
@Override
public void insertFill(MetaObject metaObject) {
    this.setFieldValByName("version", 1, metaObject);
}
```

**特别说明:**

* 支持的数据类型只有 int,Integer,long,Long,Date,Timestamp,LocalDateTime
* 整数类型下 `newVersion = oldVersion + 1```
* `newVersion` 会回写到 `entity` 中
* 仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法
* 在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!

**（4）在 MybatisPlusConfig 中注册 Bean**

```java
package com.mybatis.plus.config;

import com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor;
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("com.mybatis.plus.mapper")
public class MybatisPlusConfig {
    /**
     * 乐观锁插件
     */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
}
```

## 4、分页

MyBatis Plus自带分页插件，只要简单的配置即可实现分页功能

**（1）创建配置类**

此时可以删除主类中的 *@MapperScan* 扫描注解

```java
package com.mybatis.plus.config;

import com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor;
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("com.mybatis.plus.mapper")
public class MybatisPlusConfig {
    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

**（2）测试selectPage分页**

**测试：**最终通过page对象获取相关数据

```java
@Test
public void testSelectPage() {
    Page<User> page = new Page<>(1, 2);
    userMapper.selectPage(page, null);

    page.getRecords().forEach(System.out::println);
    System.out.println(page.getCurrent());
    System.out.println(page.getPages());
    System.out.println(page.getSize());
    System.out.println(page.getTotal());
    System.out.println(page.hasNext());
    System.out.println(page.hasPrevious());
}
```

**（3）测试selectMapsPage分页：结果集是Map**

```java
@Test
public void testSelectMapsPage() {
    Page<User> page = new Page<>(1, 5);
    IPage<Map<String, Object>> mapIPage = userMapper.selectMapsPage((Page)page, null);
    //注意：此行必须使用 mapIPage 获取记录列表，否则会有数据类型转换错误
    mapIPage.getRecords().forEach(System.out::println);
    System.out.println(page.getCurrent());
    System.out.println(page.getPages());
    System.out.println(page.getSize());
    System.out.println(page.getTotal());
    System.out.println(page.hasNext());
    System.out.println(page.hasPrevious());
}
```

## 5、逻辑删除

- 物理删除：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除数据
- 逻辑删除：假删除，将对应数据中代表是否被删除字段状态修改为“被删除状态”，之后在数据库中仍旧能看到此条数据记录

**（1）数据库中添加 deleted字段**

```java
ALTER TABLE `user` ADD COLUMN `deleted` boolean
```

**（2）实体类添加deleted *字段*

并加上 @TableLogic 注解 和 @TableField(fill = FieldFill.INSERT) 注解

```java
@TableLogic
@TableField(fill = FieldFill.INSERT)
private Integer deleted;
```

**（3）元对象处理器接口添加deleted的insert默认值**

```java
@Component
@Slf4j
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.setFieldValByName("deleted", 0, metaObject);
    }
}    
```

**（4）application.properties 加入配置**

此为默认值，如果你的默认值和mp默认的一样,该配置可无

```java
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: flag  #全局逻辑删除字段值 3.3.0开始支持，详情看下面。
      logic-delete-value: 1 	# 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

* 全局逻辑删除: begin 3.3.0

  如果公司代码比较规范，比如统一了全局都是flag为逻辑删除字段。

  使用此配置则不需要在实体类上添加 @TableLogic。

  但如果实体类上有 @TableLogic 则以实体上的为准，忽略全局。 即先查找注解再查找全局，都没有则此表没有逻辑删除。

* 若确需查找删除数据，如老板需要查看历史所有数据的统计汇总信息，请单独手写sql。

* 逻辑删除是为了方便数据恢复和保护数据本身价值等等的一种方案，但实际就是删除。

* 如果你需要再查出来就不应使用逻辑删除，而是以一个状态去表示。

  ​	如： 员工离职，账号被锁定等都应该是一个状态字段，此种场景不应使用逻辑删除。

## 6. **wapper**条件构造器

![image-20200519190242430](\img\image-20200519190242430.png)

Wrapper ： 条件构造抽象类，最顶端父类

  AbstractWrapper ： 用于查询条件封装，生成 sql 的 where 条件

​    QueryWrapper ： Entity 对象封装操作类，不是用lambda语法

​    UpdateWrapper ： Update 条件封装，用于Entity对象更新操作

  AbstractLambdaWrapper ： Lambda 语法使用 Wrapper统一处理解析 lambda 获取 column。

​    LambdaQueryWrapper ：看名称也能明白就是用于Lambda语法使用的查询Wrapper

​    LambdaUpdateWrapper ： Lambda 更新封装Wrapper



说明：

* ` boolean condition`表示该条件**是否**加入最后生成的sql中。默认为`true`
* 泛型`Param`均为`Wrapper`的子类实例(均具有`AbstractWrapper`的所有方法)
* 方法在入参中出现的`R`为泛型,在普通wrapper中是`String`,在LambdaWrapper中是**函数**(例:`Entity::getId`,`Entity`为实体类,`getId`为字段`id`的**getMethod**) 

警告:

不支持以及不赞成在 RPC 调用中把 Wrapper 进行传输

1. wrapper 很重
2. 传输 wrapper 可以类比为你的 controller 用 map 接收值(开发一时爽,维护火葬场)
3. 正确的 RPC 调用姿势是写一个 DTO 进行传输,被调用方再根据 DTO 执行相应的操作
4. 我们拒绝接受任何关于 RPC 传输 Wrapper 报错相关的 issue 甚至 pr

### QueryWrapper：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class QueryWrapperTests {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void testDelete() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                .isNull("name")
                .ge("age", 12)
                .isNotNull("email");
        int result = userMapper.delete(queryWrapper);
        System.out.println("delete return count = " + result);
    }
}
```

### lambdaQuery：

* Wrappers.<User>lambdaQuery()

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class LambdaQueryWrapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void testDelete() {
        List<User> users = userMapper.selectList(Wrappers.<User>lambdaQuery().eq(false, User::getAge, 11).eq(User::getNickName, "a"));
        users.forEach((e) -> {
            e.setNickName("a");
            userMapper.updateById(e);
        });
    }
}
```

## 7. 代码生成：

AutoGenerator 是 MyBatis-Plus 的代码生成器，通过 AutoGenerator 可以快速生成 Entity、Mapper、Mapper XML、Service、Controller 等各个模块的代码，极大的提升了开发效率。

### 约定：

* POM

```java
<properties>
    <java.version>1.8</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>

<dependencies>
    <!--mybatis-plus-boot-starter-->
     <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.3.1.tmp</version>
    </dependency>
    <!--MP Generator-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-generator</artifactId>
        <version>3.3.1.tmp</version>
    </dependency>
    <!--freemarker模板-->
    <dependency>
        <groupId>org.freemarker</groupId>
        <artifactId>freemarker</artifactId>
        <version>2.3.30</version>
    </dependency>
    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.49</version>
    </dependency>
</dependencies>
```

### 编码：

```java
// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("jobob");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/ant?useUnicode=true&useSSL=false&characterEncoding=utf8");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("密码");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("模块名"));
        pc.setParent("com.baomidou.ant");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        /*
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // 判断自定义文件夹是否需要创建
                checkDir("调用默认方法创建的目录，自定义目录用");
                if (fileType == FileType.MAPPER) {
                    // 已经生成 mapper 文件判断存在，不想重新生成返回 false
                    return !new File(filePath).exists();
                }
                // 允许生成模板文件
                return true;
            }
        });
        */
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setSuperEntityClass("你自己的父类实体,没有就不用设置!");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
        strategy.setSuperControllerClass("你自己的父类控制器,没有就不用设置!");
        // 写于父类中的公共字段
        strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}
```