# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;今天讲解一下Feign和Hystrix结合使用的场景。其实在开发中服务降级是很有必要的。如果一个服务又问题那么可能会拖垮整个微服务体系,在公司其实经历过这样的场景,因为技术问题,在公司内部的服务降级需要手动降级。详细的代码可以访问[feign-eureka-hystrix-producer](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/feign-group/feign-eureka-hystrix-producer "feign-eureka-hystrix-producer")和[feign-eureka-hystrix-consumer](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/feign-group/feign-eureka-hystrix-consumer "feign-eureka-hystrix-consumer")这两个项目。


# 服务端代码

## 依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
## 启动类

```java
package com.bobo.springcloud.learn.feigneurekahystrixproducer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class FeignEurekaHystrixProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignEurekaHystrixProducerApplication.class, args);
    }

}

```




## 配置

```yml
server:
  port: 8092
spring:
  application:
    name: feign-eureka-hystrix-producer
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
```

## 代码

```java
package com.bobo.springcloud.learn.feigneurekahystrixproducer.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author wuxiaobo
 */

@RestController
public class HelloController {

    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String hello (String name) {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return name+" world";
    }
}

```

>&nbsp;&nbsp;&nbsp;&nbsp;这里睡眠五秒就是为了让服务端请求超时。



# 客户端代码

## 依赖
```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

```


## 启动类

```java
package com.bobo.springcloud.learn.feigneurekahystrixconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
@EnableCircuitBreaker
@EnableHystrixDashboard
@EnableFeignClients
public class FeignEurekaHystrixConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignEurekaHystrixConsumerApplication.class, args);
    }

}

```

## 配置

```yml
server:
  port: 8093
spring:
  application:
    name: feign-eureka-hystrix-consumer
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
management:
  endpoints:
    web:
      exposure:
        include: "*"
feign-client-config.ConnectTimeout: 3000
feign-client-config.ReadTimeout: 3000
#开启feign中hystrix的调用
feign:
  hystrix:
    enabled: true

```

## Feign代码

```java
package com.bobo.springcloud.learn.feigneurekahystrixconsumer.feign;

import feign.Logger;
import feign.Request;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Feign的配置文件
 */
@Configuration
public class FeignConfig {
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

package com.bobo.springcloud.learn.feigneurekahystrixconsumer.feign;

import com.bobo.springcloud.learn.feigneurekahystrixconsumer.hystrix.FeignEurekaHystrixProducerHystrix;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@FeignClient(name = "feign-eureka-hystrix-producer",fallback = FeignEurekaHystrixProducerHystrix.class)
public interface FeignEurekaHystrixProducer {

    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    @ResponseBody
    String hello(@RequestParam(value = "name")String name);
}




```

## Hystrix代码

```java
package com.bobo.springcloud.learn.feigneurekahystrixconsumer.hystrix;

import com.bobo.springcloud.learn.feigneurekahystrixconsumer.feign.FeignEurekaHystrixProducer;
import org.springframework.stereotype.Component;

@Component
public class FeignEurekaHystrixProducerHystrix implements FeignEurekaHystrixProducer {
    @Override
    public String hello(String name) {
        return "error";
    }
}

```

## web代码

```java
package com.bobo.springcloud.learn.feigneurekahystrixconsumer.controller;

import com.bobo.springcloud.learn.feigneurekahystrixconsumer.feign.FeignEurekaHystrixProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    private FeignEurekaHystrixProducer feignEurekaHystrixProducer;

    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String hello(String name) {
        return feignEurekaHystrixProducer.hello(name);
    }
}

```

# 结果

>&nbsp;&nbsp;&nbsp;&nbsp;首先需要启动这两个项目,因为eureka是已经建立好的集群环境。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/8.jpg?raw=true)


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/9.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;可以看到因为服务端线程睡眠了5秒,但是在客户端配置的时间是3s,所以服务一定回存在超时,超时之后就会进入Hystrix降级的操作。

>&nbsp;&nbsp;&nbsp;&nbsp;这里稍微提一下,其实降级框架并不是只有Hystrix这一种,Hystrix是基于线程池而实现的降级。在Spring Cloud Alibaba中有一种新的框架是Sentinel。它提供的是基于QPS和平均响应时间而实现的服务降级,在之后的学习过程中会给出具体的做法。


