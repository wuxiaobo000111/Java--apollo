# FeignClient的注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FeignClient {

	//name和value对应的是一个,作为微服务的名称,是被调用方的spring.application.name属性。
	@AliasFor("name")
	String value() default "";


	@Deprecated
	String serviceId() default "";


	String contextId() default "";

	
	@AliasFor("value")
	String name() default "";

	
	String qualifier() default "";

        //手动指定@FeignClient调用的地址,比如说是:
        //@FeignClient(name = "github-client", url = "https://api.github.com", configuration = HelloFeignServiceConfig.class)
	String url() default "";

	/**
	 * 当发生404的时候，会调用docoder进行解码,否则就会抛出FeignException.
	 */
	boolean decode404() default false;

	//Feign的配置类
	Class<?>[] configuration() default {};

    //当调用接口失败或者是超时,会调用对应接口的容错逻辑,
    //faccback指定的类必须实现@FeignClient标记的接口
	Class<?> fallback() default void.class;

	//用于生成fallback类的实例,可以实现每个接口通用的容错逻辑,减少重复代码。
	Class<?> fallbackFactory() default void.class;

	//定义当前FeignClient的统一前缀。
	String path() default "";

	/**
	 * @return whether to mark the feign proxy as a primary bean. Defaults to true.
	 */
	boolean primary() default true;

}
```

# 手动创建Feign客户端

```java



@Import(FeignClientsConfiguration.class)
class FooController {

	private FooClient fooClient;

	private FooClient adminClient;

    @Autowired
	public FooController(
			Decoder decoder, Encoder encoder, Client client) {
		this.fooClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
				.target(FooClient.class, "http://PROD-SVC");
		this.adminClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
				.target(FooClient.class, "http://PROD-SVC");
    }
}
```

# Feign开启GZIP压缩



>&nbsp;&nbsp;&nbsp;&nbsp;注意一点,GZIP压缩是在客户端开启的。
```yml

feign:
  compression:
    request:
      enabled: true
#     配置压缩数据大小的下限
      min-request-size: 2048
#     配置压缩支持的MIME TYPE
      mime-types: text/xml,application/xml,application/json
#   是否开启GZIP压缩
    response:
      enabled: true
```


>&nbsp;&nbsp;&nbsp;&nbsp; 启动feign-demo项目,然后去访问接口。这里在有的书上或者博客中提到会有中文乱码的问题,在这里我去请求的时候并没有发现中文乱码,猜测可能与spring cloud的版本有关,如果遇到中文乱码的问题,则将接口的返回值设置为byte[]类型即可。


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/12.jpg?raw=true)



# Feign支持属性文件配置

>&nbsp;&nbsp;&nbsp;&nbsp; 


```yml
feign:
  compression:
    request:
      enabled: true
#     配置压缩数据大小的下限
      min-request-size: 2048
#     配置压缩支持的MIME TYPE
      mime-types: text/xml,application/xml,application/json
#   是否开启GZIP压缩
    response:
      enabled: true
  client:
    config:
#    相当于FeignClient注解中的name或者是value属性
      feignName:
#      连接超时时间
        connectTimeOut: 5000
#      读超时时间设置
        readTimeOut: 5000
#      日志级别
        loggerLevel: full
#      错误解码器
        errorDecoder: com.bobo.SimpleErrorDecoder
#      配置重试
        retryer: com.bobo.SimpleRetryer
        decode404: false
#        编码器
        encoder: com.bobo.SimpleEncoder
#        解码器
        decoder: com.bobo.SimpleDecoder
#        contract设置
        contract: com.bobo.SimpleContract

```


# Feign的日志

```java
package com.bobo.springcloud.feign.feigneurekaconsumer.config;

import feign.Logger;
import feign.Request;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HelloFeignServiceConfig {


    /**
     * 连接超时时间
     */
    @Value("${feign-client-config.ConnectTimeout}")
    private int connectTimeout;

    /**
     *响应时间
     */
    @Value("${feign-client-config.ReadTimeout}")
    private int readTimeout;

    @Bean
    public Request.Options options() {
        return new Request.Options(connectTimeout, readTimeout);
    }


    /**
     *
     * Logger.Level 的具体级别如下：
         NONE：不记录任何信息
         BASIC：仅记录请求方法、URL以及响应状态码和执行时间
         HEADERS：除了记录 BASIC级别的信息外，还会记录请求和响应的头信息
         FULL：记录所有请求与响应的明细，包括头信息、请求体、元数据
     * @return
     */
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }

}

```

# Feign默认Client的替换


>&nbsp;&nbsp;&nbsp;&nbsp;Feign在默认的情况下使用的是JDK原生的URLConnection发送HTTP请求,没有使用连接池,但是对于每个地址都有一个长连接,使用的是HTTP的persistence connection。可以使用其他的http client客户端对原始客户端进行替代,通过设置连接池,超时时间对服务进行调优。

## 使用HTTP client替换Feign默认的Client

### 引入依赖

```xml

        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
        </dependency>
        <dependency>
            <groupId>com.netflix.feign</groupId>
            <artifactId>feign-httpclient</artifactId>
            <version>8.18.0</version>
        </dependency>


```

###  配置文件

```yml
feign:
  httpclient:
    enabled: true
```

## 使用OKHttp替换Feign默认的client

### pom文件

```xml
 <!-- https://mvnrepository.com/artifact/io.github.openfeign/feign-okhttp -->
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-okhttp</artifactId>
            <version>9.2.0</version>
        </dependency>
```

### 修改配置文件
```yml
feign:
  httpclient:
    enabled: false
  okhttp:
    enabled: true
```

### 配置文件

```java
package com.bobo.springcloud.feign.feigndemo.config;

import feign.Feign;
import okhttp3.ConnectionPool;
import org.springframework.boot.autoconfigure.AutoConfigureBefore;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.cloud.openfeign.FeignAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;

@ConditionalOnClass(Feign.class)
@AutoConfigureBefore(FeignAutoConfiguration.class)
@Configuration
public class FeignOkHttpConfig {

    @Bean
    public okhttp3.OkHttpClient okHttpClient() {
        return new okhttp3.OkHttpClient().newBuilder()
                .connectTimeout(60,TimeUnit.SECONDS)
                .readTimeout(60,TimeUnit.SECONDS)
                .writeTimeout(60,TimeUnit.SECONDS)
                .retryOnConnectionFailure(true)
                .connectionPool(new ConnectionPool()).build();
    }
}

```

# Feign的文件上传

>&nbsp;&nbsp;&nbsp;&nbsp;可以使用Feign的子项目feign-form实现文件上传的功能。具体项目地址参考[feign-file-producer](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/feign-group/feign-file-producer "feign-file-producer")和[feign-file-consumer](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/feign-group/feign-file-consumer "feign-file-consumer")。两个项目,代码比较多,这里就不多做介绍了,直接去访问项目,之中有源代码。




