# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这篇文章介绍一下Feign和eureka结合使用,实现HTTP调用的实例。详细的代码可以访问[feign-eureka-producer](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/feign-group/feign-eureka-producer "feign-eureka-producer")和[feign-eureka-consumer](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/feign-group/feign-eureka-consumer "feign-eureka-consumer")这两个项目。

# feign-eureka-producer 

## pom依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.bobo.springcloud.feign</groupId>
    <artifactId>feign-eureka-producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>feign-eureka-producer</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

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

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

>&nbsp;&nbsp;&nbsp;&nbsp;这里引入eureka,是为了将接口服务注入到eureka中去。

## 启动类
```java
package com.bobo.springcloud.feign.feigneurekaproducer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class FeignEurekaProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignEurekaProducerApplication.class, args);
    }

}

```

>&nbsp;&nbsp;&nbsp;&nbsp;@EnableDiscoveryClient通过这个注解,可以将服务注册到eureka中去。

## 接口文件

```java
package com.bobo.springcloud.feign.feigneurekaproducer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author wuxiaobo
 */
@RestController
public class HelloController {

    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String hello(String name) {
        return name+" hello";
    }
}

```

## 配置文件


```yml

server:
  port: 8083
spring:
  application:
    name: feign-eureka-producer
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

>&nbsp;&nbsp;&nbsp;&nbsp;这里配置的是最小的配置单元。

# feign-eureka-consumer

## pom依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.bobo.springcloud.feign</groupId>
    <artifactId>feign-eureka-consumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>feign-eureka-consumer</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

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
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

## 启动类
```
package com.bobo.springcloud.feign.feigneurekaconsumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class FeignEurekaConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignEurekaConsumerApplication.class, args);
    }

}

```

>&nbsp;&nbsp;&nbsp;&nbsp;EnableDiscoveryClient是将服务注册到eureka中去。@EnableFeignClients声明是Feign的客户端。

# Feign的配置文件
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

## 接口
```java
package com.bobo.springcloud.feign.feigneurekaconsumer.service;

import com.bobo.springcloud.feign.feigneurekaconsumer.config.HelloFeignServiceConfig;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * @author wuxiaobo
 */
@FeignClient(name = "feign-eureka-producer", configuration = HelloFeignServiceConfig.class)
public interface HelloService {

    /**
     * @param name
     * @return
     */
    @RequestMapping(value = "/hello", method = RequestMethod.GET, produces = "application/json;charset=UTF-8")
    String searchRepo(@RequestParam("name") String name);

}

```
>&nbsp;@FeignClient中的name和被调用方的spring.application.name对应,接口名可以随意。在公司是又规范的。这个注意。

## 调用方
```java
package com.bobo.springcloud.feign.feigneurekaconsumer.controller;


import com.bobo.springcloud.feign.feigneurekaconsumer.service.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloFeignController {

    @Autowired
    private HelloService helloService;

    // 服务消费者对位提供的服务
    @GetMapping(value = "/name")
    public String sayHello(@RequestParam("name") String name) {
        return helloService.searchRepo(name);
    }

}

```

## 配置文件
```yml

feign-client-config:
  ConnectTimeout: 3000
  ReadTimeout: 3000
server:
  port: 8084
spring:
  application:
    name: feign-eureka-consumer
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

>&nbsp;&nbsp;&nbsp;&nbsp;至此,所有的代码都已经完毕。

# 结果。

>&nbsp;&nbsp;&nbsp;&nbsp;因为是将服务注册到了eureka中去,(Spring Cloud的注册中心并不是只有eureka一种,比较常用的还有consul,在之后的篇幅中会具体讲到。)所以在启动项目之前,需要先启动eureka-server项目。



![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/2.jpg?raw=true)



![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/3.jpg?raw=true)