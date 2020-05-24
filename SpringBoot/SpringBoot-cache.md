## 一、JSR107:

Java Caching定义了5个核心接口，分别是**CachingProvider**, **CacheManager**, **Cache**, **Entry** 和 **Expiry**。

•**CachingProvider**定义了创建、配置、获取、管理和控制多个**CacheManager**。一个应用可以在运行期访问多个CachingProvider。

•**CacheManager**定义了创建、配置、获取、管理和控制多个唯一命名的**Cache**，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。

•**Cache**是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。

•**Entry**是一个存储在Cache中的key-value对。

•**Expiry** 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

## 二、Spring缓存抽象

Spring从3.1开始定义了org.springframework.cache.Cache

和org.springframework.cache.CacheManager接口来统一不同的缓存技术；

并支持使用JCache（JSR-107）注解简化我们开发；



•Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；

•Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等；

•

•每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。

•使用Spring缓存抽象时我们需要关注以下两点；

1、确定方法需要被缓存以及他们的缓存策略

2、从缓存中读取之前缓存存储的数据

## 三、几个重要概念&缓存注解

| **Cache**              | **缓存接口，定义缓存操作。实现有：****RedisCache****、****EhCacheCache****、****ConcurrentMapCache****等** |
| ---------------------- | ------------------------------------------------------------ |
| **CacheManager**       | **缓存管理器，管理各种缓存（****Cache****）组件**            |
| **@Cacheable**         | **主要针对方法配置，能够根据方法的请求参数对其结果进行缓存** |
| **@****CacheEvict**    | **清空缓存**                                                 |
| **@****CachePut**      | **保证方法被调用，又希望结果被缓存。**                       |
| **@****EnableCaching** | **开启基于注解的缓存**                                       |
| **keyGenerator**       | **缓存数据时****key****生成策略**                            |
| **serialize**          | **缓存数据时****value****序列化策略**                        |



![image-20200521232703297](images\image-20200521232703297.png)

![image-20200521232753932](images\image-20200521232753932.png)

## 四、SpringBoot Cache



### 1、引入spring-boot-starter-cache模块

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### 2、@EnableCaching开启缓存

```
@SpringBootApplication
@EnableCaching
public class Springboot01CacheApplication {

   public static void main(String[] args) {
      SpringApplication.run(Springboot01CacheApplication.class, args);
   }
}
```

### 3、使用缓存注解

* **@Cacheable**：**主要针对方法配置，能够根据方法的请求参数对其结果进行缓存**

```
/**
 * 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
 * CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
 *

 *
 * 原理：
 *   1、自动配置类；CacheAutoConfiguration
 *   2、缓存的配置类
 *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
 *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
 *   org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
 *   3、哪个配置类默认生效：SimpleCacheConfiguration；
 *
 *   4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager
 *   5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；
 *
 *   运行流程：
 *   @Cacheable：
 *   1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
 *      （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
 *   2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
 *      key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
 *          SimpleKeyGenerator生成key的默认策略；
 *                  如果没有参数；key=new SimpleKey()；
 *                  如果有一个参数：key=参数的值
 *                  如果有多个参数：key=new SimpleKey(params)；
 *   3、没有查到缓存就调用目标方法；
 *   4、将目标方法返回的结果，放进缓存中
 *
 *   @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
 *   如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
 *
 *   核心：
 *      1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
 *      2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
 *
 *
 *   几个属性：
 *      cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
 *
 *      key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
 *              编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0]
 *              getEmp[2]
 *
 *      keyGenerator：key的生成器；可以自己指定key的生成器的组件id
 *              key/keyGenerator：二选一使用;
 *
 *
 *      cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
 *
 *      condition：指定符合条件的情况下才缓存；
 *              ,condition = "#id>0"
 *          condition = "#a0>1"：第一个参数的值》1的时候才进行缓存
 *
 *      unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
 *              unless = "#result == null"
 *              unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
 *      sync：是否使用异步模式
 * @param id
 * @return
 *
 */
@Cacheable(value = {"emp"}/*,keyGenerator = "myKeyGenerator",condition = "#a0>1",unless = "#a0==2"*/)
public Employee getEmp(Integer id){
    System.out.println("查询"+id+"号员工");
    Employee emp = employeeMapper.getEmpById(id);
    return emp;
}
```

* **@CacheEvict**：**清空缓存**

```java
/**
 * @CacheEvict：缓存清除
 *  key：指定要清除的数据
 *  allEntries = true：指定清除这个缓存中所有的数据
 *  beforeInvocation = false：缓存的清除是否在方法之前执行
 *      默认代表缓存清除操作是在方法执行之后执行;如果出现异常缓存就不会清除
 *
 *  beforeInvocation = true：
 *      代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除
 *
 *
 */
@CacheEvict(value="emp",beforeInvocation = true/*key = "#id",*/)
public void deleteEmp(Integer id){
    System.out.println("deleteEmp:"+id);
    //employeeMapper.deleteEmpById(id);
    int i = 10/0;
}
```

* **@CachePut**：**保证方法被调用，又希望结果被缓存。**

```java
/**
 * @CachePut：既调用方法，又更新缓存数据；同步更新缓存
 * 修改了数据库的某个数据，同时更新缓存；
 * 运行时机：
 *  1、先调用目标方法
 *  2、将目标方法的结果缓存起来
 *
 * 测试步骤：
 *  1、查询1号员工；查到的结果会放在缓存中；
 *          key：1  value：lastName：张三
 *  2、以后查询还是之前的结果
 *  3、更新1号员工；【lastName:zhangsan；gender:0】
 *          将方法的返回值也放进缓存了；
 *          key：传入的employee对象  值：返回的employee对象；
 *  4、查询1号员工？
 *      应该是更新后的员工；
 *          key = "#employee.id":使用传入的参数的员工id；
 *          key = "#result.id"：使用返回后的id
 *             @Cacheable的key是不能用#result
 *      为什么是没更新前的？【1号员工没有在缓存中更新】
 *
 */
//@CachePut(/*value = "emp",*/key = "#result.id")
public Employee updateEmp(Employee employee){
    System.out.println("updateEmp:"+employee);
    employeeMapper.updateEmp(employee);
    return employee;
}
```

* @Caching 定义复杂的缓存规则

```java
// @Caching 定义复杂的缓存规则
@Caching(
     cacheable = {
         @Cacheable(/*value="emp",*/key = "#lastName")
     },
     put = {
         @CachePut(/*value="emp",*/key = "#result.id"),
         @CachePut(/*value="emp",*/key = "#result.email")
     }
)
public Employee getEmpByLastName(String lastName){
    return employeeMapper.getEmpByLastName(lastName);
}
```

### 4、缓存配置：

```java
@Configuration
public class MyCacheConfig {
	//缓存数据时key生成策略
    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return new KeyGenerator(){
            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName()+"["+ Arrays.asList(params).toString()+"]";
            }
        };
    }
}
```

### 4、切换为Redis缓存

​	redis 默认存储对象需要序列化（Serializable）

* POM

```JAVA
<!--默认为jedis客户端-->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

* yml

```JAVA
spring:
  redis:
    host: luliang888.top
```

* 编码测试：

```java
    @Autowired
   StringRedisTemplate stringRedisTemplate;  //操作k-v都是字符串的
   @Autowired
   RedisTemplate redisTemplate;  //k-v都是对象的
   @Autowired
   RedisTemplate<Object, Employee> empRedisTemplate;

   /**
    * Redis常见的五大数据类型
    *  String（字符串）、List（列表）、Set（集合）、Hash（散列）、ZSet（有序集合）
    *  stringRedisTemplate.opsForValue()[String（字符串）]
    *  stringRedisTemplate.opsForList()[List（列表）]
    *  stringRedisTemplate.opsForSet()[Set（集合）]
    *  stringRedisTemplate.opsForHash()[Hash（散列）]
    *  stringRedisTemplate.opsForZSet()[ZSet（有序集合）]
    */
   @Test
   public void test01(){
      //给redis中保存数据
       stringRedisTemplate.opsForValue().append("msg","hello");
      String msg = stringRedisTemplate.opsForValue().get("msg");
      System.out.println(msg);

      stringRedisTemplate.opsForList().leftPush("mylist","1");
      stringRedisTemplate.opsForList().leftPush("mylist","2");
   }

   //测试保存对象
   @Test
   public void test02(){
      Employee empById = employeeMapper.getEmpById(1);
      //默认如果保存对象，使用jdk序列化机制，序列化后的数据保存到redis中
      //redisTemplate.opsForValue().set("emp-01",empById);
      //1、将数据以json的方式保存
       //(1)自己将对象转为json
       //(2)redisTemplate默认的序列化规则；改变默认的序列化规则；
      empRedisTemplate.opsForValue().set("emp-01",empById);
   }
```



* Redis自动配置：MyRedisConfig

```
@Configuration
public class MyRedisConfig {
    //改变默认序列化规则
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }
    //CacheManagerCustomizers可以来定制缓存的一些规则
    @Primary  //将某个缓存管理器作为默认的
    @Bean
    public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        //key多了一个前缀

        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);
    
        return cacheManager;
    }
} 
```

* 编码测试

```java
@Service
public class DeptService {

    @Autowired
    DepartmentMapper departmentMapper;

    @Qualifier("deptCacheManager")
    @Autowired
    RedisCacheManager deptCacheManager;


    /**
     *  缓存的数据能存入redis；
     *  第二次从缓存中查询就不能反序列化回来；
     *  存的是dept的json数据;CacheManager默认使用RedisTemplate<Object, Employee>操作Redis
     *
     *
     * @param id
     * @return
     */
//    @Cacheable(cacheNames = "dept",cacheManager = "deptCacheManager")
//    public Department getDeptById(Integer id){
//        System.out.println("查询部门"+id);
//        Department department = departmentMapper.getDeptById(id);
//        return department;
//    }

    // 使用缓存管理器得到缓存，进行api调用
    public Department getDeptById(Integer id){
        System.out.println("查询部门"+id);
        Department department = departmentMapper.getDeptById(id);

        //获取某个缓存
        Cache dept = deptCacheManager.getCache("dept");
        dept.put("dept:1",department);

        return department;
    }
    
}
```

