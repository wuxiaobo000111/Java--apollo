# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这里给出cloud中如何使用zipkin。<font color=red>这里明确标出,当时我在搭建这个项目的使用使用的是cloud 2.1.5 版本的。在zipkin中可以指定spring.zipkin.sender.type=web,但是不能使用。这里我也不知道是为什么。必须指定这个type是rabbit或者是kafka。这里我使用的是rabbitmq。<font color=black>这里给出我的项目的两个地址[sleuth-eureka-demo-client](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/trace-group/sleuth-eureka-demo-client  "sleuth-eureka-demo-client")和[sleuth-eureka-demo-server](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/trace-group/sleuth-eureka-demo-server  "sleuth-eureka-demo-server")。
&nbsp;&nbsp;&nbsp;&nbsp;<font color=red>这里还要注意的两点是在cloud 2.X版本之后,zipkin官方提供了zipkin服务运行的jar包。所以可以去官网上下。这里还有一份关于rabbitmq在windows环境下的安装教程。参考链接地址是https://blog.csdn.net/weixin_42931208/article/details/81735324。
&nbsp;&nbsp;&nbsp;&nbsp;<font color=black>万事俱备,只欠东风。下面就让我们来看一下如何实现zipkin.

# sleuth-eureka-demo-server

## 依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
        </dependency>
    </dependencies>
```

## 配置文件

```yml
server:
  port: 8120
spring:
  application:
    name: sleuth-eureka-demo-server
  zipkin:
    sender:
      type: rabbit
    service:
      name: sleuth-eureka-demo-server
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
  zipkin:
    base-url: http://localhost:9411/
```

##  启动类

```java
package com.bobo.springcloud.learn.sleutheurekademoserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;


@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class SleuthEurekaDemoServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SleuthEurekaDemoServerApplication.class, args);
    }
}

```

## 接口文件

```java
package com.bobo.springcloud.learn.sleutheurekademoserver.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping(value = "/hello")
    public String hello (@RequestParam String name) {
        return "hello, "+ name;
    }
}

```

# sleuth-eureka-demo-client

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
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.4</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```

## 配置文件

```yml
server:
  port: 8121
spring:
  application:
    name: sleuth-eureka-demo-client
  sleuth:
    sampler:
      probability: 1.0
  zipkin:
    sender:
      type: rabbit
    service:
      name: sleuth-eureka-demo-client
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
  zipkin:
    base-url: http://localhost:9411/
```

## 启动类

```java
package com.bobo.springcloud.learn.sleutheurekademoclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class SleuthEurekaDemoClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SleuthEurekaDemoClientApplication.class, args);
    }

}

```

## feign

```java
package com.bobo.springcloud.learn.sleutheurekademoclient.feign;

import feign.Logger;
import feign.Request;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
 * Feign的配置文件
 */
@Configuration
public class FeignConfig {

    @Bean
    public Request.Options options() {
        return new Request.Options(3000, 3000);
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

package com.bobo.springcloud.learn.sleutheurekademoclient.feign;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * @author wuxiaobo
 */
@FeignClient(name = "sleuth-eureka-demo-server", configuration = FeignConfig.class)
public interface HelloService {

    /**
     * @param name
     * @return
     */
    @RequestMapping(value = "/hello", method = RequestMethod.GET, produces = "application/json;charset=UTF-8")
    String hello(@RequestParam("name") String name);

}
```

## controller

```java
package com.bobo.springcloud.learn.sleutheurekademoclient.controller;

import com.bobo.springcloud.learn.sleutheurekademoclient.feign.HelloService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Future;

@RestController
public class HelloController {

    private static final Logger log = LoggerFactory.getLogger(HelloController.class);
    @Autowired
    private HelloService helloService;

    @Autowired
    private ExecutorService executorService;

    @GetMapping(value = "/name")
    public String hello(@RequestParam String name) throws Exception {
        Future future = executorService.submit(() -> {
            log.info("client sent. 进入子线程, 参数: {}",name);
            String result = helloService.hello(name);
            return result;
        });
        String result = (String) future.get();
        log.info("client received. 返回主线程, 结果: {}",result);
        return result;
    }
}

```

## 结果

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/67.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/68.jpg?raw=true)
