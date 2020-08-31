# FeignClient里如何进行重试（Retry）和超时（timeout）配置

FeigninClient的默认connectTimeout为10s，readTimeout为60。仅设置超时可能不会立即生效，因为默认重试次数为5次。 因此，如果想要快速失败，则必须同时自定义超时和重试的参数，并应确保反向代理。 例如，nginx的proxy_connect_timeout和proxy_read_timeout必须大于feign的配置才能生效。 否则，nginx的504网关超时仍然会被外部用户感知，并且无法实现回滚效果。

* 配置类

```
package cn.com.hopson.hopsonone.park.js.conf;

import feign.Request;
import feign.Retryer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;


@Configuration
public class FeignConfig {


    /**
     * 配置请求重试
     */
    @Bean
    public Retryer feignRetryer() {
        return new Retryer.Default(200, TimeUnit.SECONDS.toMillis(10), 10);
    }


    /**
     * 设置请求超时时间
     * 默认
     * public Options() {
     * this(10 * 1000, 60 * 1000);
     * }
     */
    @Bean
    Request.Options feignOptions() {
        return new Request.Options(60 * 1000, 60 * 1000);
    }


    /**
     * 打印请求日志
     * <p>
     * NONE: 不记录任何信息
     * BASIE:仅记录请求方法，URL以及响应状态码和执行时间
     * HEADERS:除了记录BASIE级别得信息之外，还会记录请求和响应得头信息
     * FULL：记录所有请求与响应得明细，包括头信息，请求体，元数据等。
     *
     * @return
     */
    @Bean
    public feign.Logger.Level multipartLoggerLevel() {
        return feign.Logger.Level.FULL;
    }

}
```

* 注解指定配置文件

```
@FeignClient(name = "hopsonone-park-search", configuration = FeignConfig.class)
```