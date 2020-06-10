# SpringBoot整合ES

SpringBoot默认支持两种技术来和ES交互

## 1、Jest

* 默认不生效,需要导入jest的工具包（io.searchbox.client.JestClient）

```java
		<dependency>
			<groupId>io.searchbox</groupId>
			<artifactId>jest</artifactId>
			<version>6.3.1</version>
		</dependency>
```

* yml配置：

```
spring.elasticsearch.jest.uris=http://118.24.44.169:9200
```

* 编码：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot03ElasticApplicationTests {

   @Autowired
   JestClient jestClient;

   @Test
   public void contextLoads() {
      //1、给Es中索引（保存）一个文档；
      Article article = new Article();
      article.setId(1);
      article.setTitle("好消息");
      article.setAuthor("zhangsan");
      article.setContent("Hello World");

      //构建一个索引功能
      Index index = new Index.Builder(article).index("atguigu").type("news").build();

      try {
         //执行
         jestClient.execute(index);
      } catch (IOException e) {
         e.printStackTrace();
      }
      //测试：http://118.24.44.169:9200/atguigu/news/1
   }

   //测试搜索
   @Test
   public void search(){

      //查询表达式
      String json ="{\n" +
            "    \"query\" : {\n" +
            "        \"match\" : {\n" +
            "            \"content\" : \"hello\"\n" +
            "        }\n" +
            "    }\n" +
            "}";

      //更多操作：https://github.com/searchbox-io/Jest/tree/master/jest
      //构建搜索功能
      Search search = new Search.Builder(json).addIndex("atguigu").addType("news").build();

      //执行
      try {
         SearchResult result = jestClient.execute(search);
         System.out.println(result.getJsonString());
      } catch (IOException e) {
         e.printStackTrace();
      }
   }
}   
```



## 2、SpringData ElasticSearch

1）、传输客户端，TransportClient与RestClient。注意：从4.0版开始不推荐使用TransportClient，请改用RestClient。

```
https://github.com/spring-projects/spring-data-elasticsearch
```

3）、ES版本与SpringBootData版本关系查看：

```
https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#preface.metadata
```

![image-20200524233049853](C:\Users\luliang\Desktop\Blog\ElasticSearch\img\image-20200524233049853.png)

* POM

```java
<!--SpringBoot默认使用SpringData ElasticSearch模块进行操作-->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

* YML

```java
spring.data.elasticsearch.client.reactive.endpoints=luliang888.top:9200
```

* 配置

  从4.0版开始不推荐使用TransportClient，请改用RestClient

```java
@Configuration
public class RestClientConfig extends AbstractElasticsearchConfiguration {

    @Value("${spring.data.elasticsearch.client.reactive.endpoints}")
    private String url;
    @Override
    @Bean
    public RestHighLevelClient elasticsearchClient() {
        System.out.println("*********" + url);
        final ClientConfiguration clientConfiguration = ClientConfiguration.builder()
                .connectedTo(url)
                .build();

        return RestClients.create(clientConfiguration).rest();
    }
}
```

* 编码



