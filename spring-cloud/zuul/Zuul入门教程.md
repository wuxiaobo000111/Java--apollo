# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这里给出一个zuul的最简单的例子。大概涉及到了两个项目。[zuul-eureka-server-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/zuul-group/zuul-eureka-server-demo "zuul-eureka-server-demo")和[zuul-eureka-server-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/zuul-group/zuul-eureka-server-demo "zuul-eureka-server-demo")


# 接口

# POM依赖

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

## 配置文件

```yml
server:
  port: 8103
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
spring:
  application:
    name: zuul-eureka-client-demo


```

## 启动类

```java
package com.bobo.springcloud.learn.zuuleurekaclientdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ZuulEurekaClientDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulEurekaClientDemoApplication.class, args);
    }

}

```

## 接口

```java
package com.bobo.springcloud.learn.zuuleurekaclientdemo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping(value = "/hello")
    public String  hello (String name) {
        return "hello, "+ name;
    }
}

```


# zuul服务

## pom依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

```

## 配置文件

```yml
server:
  port: 8102
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
spring:
  application:
    name: zuul-eureka-server-demo
zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-client-demo


```

>&nbsp;&nbsp;&nbsp;&nbsp;zuul.routes:指定了路由规则。这里配置的意思是说在访问路径中/client/的所有请求都会映射到zuul-eureka-client-demo这个服务上。


## 启动类

```java

package com.bobo.springcloud.learn.zuuleurekaserverdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableDiscoveryClient
@EnableZuulProxy
public class ZuulEurekaServerDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulEurekaServerDemoApplication.class, args);
    }

}

```
# 结果

>&nbsp;&nbsp;&nbsp;&nbsp;启动两个项目。然后通过访问http://127.0.0.1:8102/client/hello?name=wuxiaobo。就会获取到自己想要的结果。

