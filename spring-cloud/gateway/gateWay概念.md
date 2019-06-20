# GateWay概述

&nbsp;&nbsp;&nbsp;&nbsp;Spring Cloud Gateway是Spring官方基于Spring 5.0，Spring Boot 2.0和Project Reactor等技术开发的网关，Spring Cloud Gateway旨在为微服务架构提供一种简单而有效的统一的API路由管理方式。Spring Cloud Gateway作为Spring Cloud生态系统中的网关，目标是替代Netflix ZUUL，其不仅提供统一的路由方式，并且基于Filter链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。

# GateWay的核心概念

&nbsp;&nbsp;&nbsp;&nbsp;网关简单的说就是提供一个对外统一的API入口和出口，统管企业对外的所有API出口。一般来说，网关对外暴露的URL或者接口信息，我们统称之为路由信息。如果研发过网关中间件，或者使用或了解过ZUUL的，网关的核心肯定是Filter以及Filter Chain(Filter责任链)。Spring Cloud Gateway也具有路由信息和Filter。下面介绍一下Spring Cloud gateway中最重要的几个概念:

&nbsp;&nbsp;&nbsp;&nbsp;路由(route):路由网关的基本构建块。它由一个ID、一个目的URI、一组谓词和一组过滤器定义。如果聚合谓词为真，则路由匹配。

&nbsp;&nbsp;&nbsp;&nbsp;断言(predicate): java 8中的断言函数。Spring Cloud Gateway中的断言函数输入类型是Spring 5.0框架中的ServerWebExchange。Spring Cloud Gateway中的断言函数允许开发者去定义匹配来自于http request中的任何信息，比如请求头和参数等。

&nbsp;&nbsp;&nbsp;&nbsp;过滤器(filter):一个标准的Spring webFilter。Spring Cloud Gateway中的Filter分为两种类型的Filter，分别是Gateway Filter和Global Filter.网关 Filter实例是由Spring 框架中的网关Filter的特殊工厂构造。request在转发到目前服务之前，response在返回到调用端之前都可以被修改或者自定义。

# 代码示例

这里给出两个项目,一个是基于java代码实现的网关,一个是基于配置实现的网关。项目地址如下所示:https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/gateway-group/gateway-demo-by-java 和 https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/gateway-group/gateway-demo-by-config

## 基于java代码实现的网关配置

### 依赖

```xml

 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

```


### 配置文件

```yml

server:
  port: 8136
spring:
  application:
    name: gateway-demo-by-java
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.133:8761/eureka/,http://192.168.88.133:8760/eureka/


logging: ## Spring Cloud Gateway的日志配置
  level:
    org.springframework.cloud.gateway: TRACE
    org.springframework.http.server.reactive: DEBUG
    org.springframework.web.reactive: DEBUG
    reactor.ipc.netty: DEBUG


management:
  endpoints:
    web:
      exposure:
        include: '*'

```


### 启动类

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
@EnableDiscoveryClient
public class GatewayDemoByJavaApplication {

    /**
     * 基本的转发
     * 当访问http://localhost:8080/jd
     * 转发到http://jd.com
     * @param builder
     * @return
     */

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                //basic proxy
                .route(r ->r.path("/jd")
                        .uri("http://jd.com:80/").id("jd_route")
                ).build();
    }

    public static void main(String[] args) {
        SpringApplication.run(GatewayDemoByJavaApplication.class, args);
    }

}

```


## 基于配置实现的网关服务


### 依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>


```

### 配置


```yml
server:
  port: 8137
spring:
  application:
    name: gateway-demo-by-config
  cloud:
    gateway:
      routes:
      - id: baidu_route
        uri: http://www.baidu.com
        predicates:
        - Path=/baidu

eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.133:8761/eureka/,http://192.168.88.133:8760/eureka/
logging: ## Spring Cloud Gateway的日志配置
  level:
    org.springframework.cloud.gateway: TRACE
    org.springframework.http.server.reactive: DEBUG
    org.springframework.web.reactive: DEBUG
    reactor.ipc.netty: DEBUG

management:
  endpoints:
    web:
      exposure:
        include: '*'
  security:
    enabled: false



```

### 启动类

```java
package com.bobo.springcloud.learn.gatewaydemobyconfig;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class GatewayDemoByConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayDemoByConfigApplication.class, args);
    }

}


```


### 结果

通过访问http://localhost:8137/baidu可以访问百度。