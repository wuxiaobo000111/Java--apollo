# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这里主要给出关于基于consul服务发现与注册的代码示例,这里给出了两个项目,两个项目的地址分别是[consul-demo-server](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/consul-group/consul-demo-server "consul-demo-serve")和[consul-demo-client](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/consul-group/consul-demo-client "consul-demo-client")。

# 服务注册

## pom依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-config</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
    </dependencies>
```

## 配置文件

```yml
server:
  port: 8110
spring:
  application:
    name: consul-demo-server
  cloud:
    consul:
      discovery:
        service-name: consul-demo-server
      host: localhost
      port: 8500
```
## 启动类

``java
package com.bobo.springcloud.learn.consuldemoserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ConsulDemoServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulDemoServerApplication.class, args);
    }

}


```

## 接口文档

```java
package com.bobo.springcloud.learn.consuldemoserver.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {


    /**
     * 提供 sayHello 服务:根据对方传来的名字 XX，返回:hello XX
     * @return String
     */
    @GetMapping("/sayHello")
    public String sayHello(String name){
        return "hello," + name;
    }
}

```

# 客户端

## POM依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-ribbon</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
    </dependencies>

```

## 配置文件

```yml
server:
  port: 8111
spring:
  application:
    name: consul-demo-client
  cloud:
    consul:
      discovery:
        service-name: consul-demo-client
      host: localhost
      port: 8500
feign-client-config.ConnectTimeout: 3000
feign-client-config.ReadTimeout: 3000
```

## 启动类

```java
package com.bobo.springcloud.learn.consuldemoclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class ConsulDemoClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulDemoClientApplication.class, args);
    }

}


```

## feign相关

```java

package com.bobo.springcloud.learn.consuldemoclient.feign;

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

package com.bobo.springcloud.learn.consuldemoclient.feign;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

/** 使用 openfeign 组件，调用远程服务 */
@FeignClient(name = "consul-demo-server",configuration = FeignConfig.class)
public interface HelloService {
    @RequestMapping(value = "/sayHello",method = RequestMethod.GET)
    String sayHello(@RequestParam("name") String name);
}


```

## 接口

```java
package com.bobo.springcloud.learn.consuldemoclient.controller;

import com.bobo.springcloud.learn.consuldemoclient.feign.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    private HelloService helloService;

    @GetMapping(value = "/hello")
    public String hello (String name ) {
        return helloService.sayHello(name);
    }
}

```

# 结果

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后先后启动两个项目，结果如下图所示:


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/40.jpg?raw=true)


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/41.jpg?raw=true)
