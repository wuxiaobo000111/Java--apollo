# 概述

&nbsp;&nbsp;&nbsp;&nbsp;通过上面的入门案例配置路由转发,我们快速学习Spring Cloud Gateway并入门。SpringCloud Gateway的路由匹配的功能是以Spring WebFlux中的Handler Mapping为基础实现的。Spring Cloud Gateway也是由许多的路由断言工厂组成的。当Http Request请求进入SpringCloud Gateway的时候,网关中的路由断言工厂会根据配置的路由规则,对Htp Request请求进行断言匹配。匹配成功则进行下一步处理,否则断言失败直接返回错误信息。下面我们介绍一下Spring Cloud Gateway中经常使用的路由断言工厂.


# 断言的类型


注意:所有的断言的项目都在 https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/gateway-group/gateway-prodicate-demo 这个路径之下。

## 依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
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
    </dependencies>

````
## After路由断言工厂

After Route Predicate Factory 中会取一个UTC时间格式的时间参数,当请求进来的时候在配置的UTC时间之后,就会匹配成功,否则就会匹配失败。配置如下所示:
```yml
server:
  port: 8138
spring:
  application:
    name: gateway-prodicate-demo
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: http://www.baidu.com
        predicates:
        # 在这里时间之后的都会成功,否则都不会成功，after断言工厂
        - After=2020-06-16T11:50:10.785+08:00[Asia/Shanghai]

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

这里我们配置的是在2020年之后,所以如果请求了之后并不会成功。
如果修改为2019的时间,就可以成功。



## Before路由断言工厂

Before路由断言工厂会取一个UTC时间格式的时间参数,当请求过来的当前时间在路由断言工厂之前就会成功匹配,否则就会匹配不成功。

如下所示,我们在配置文件中新增配置。

```yml
spring:
  application:
    name: gateway-prodicate-demo
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: http://www.baidu.com
        predicates:
        # 在这里时间之后的都会成功,否则都不会成功，after断言工厂
        - After=2020-06-16T11:50:10.785+08:00[Asia/Shanghai]
      - id: before_route
        uri: http://www.taobao.com
        predicates:
          - Path=/taobao
          - Before=2020-06-16T11:50:10.785+08:00[Asia/Shanghai]
```

淘宝的路径是新加的,配置的时间是2020年之前,所以可以访问的到,如果修改为2018年之前,就会访问不到。


## Between路由断言工厂

Between路由断言工厂会取一个UTC时间格式的时间参数,当请求进来的当前时间在配置的UTC时间工厂之前会成功匹配,否则就不能成功匹配。

我们在项目中新增如下配置:

```yml

spring:
  application:
    name: gateway-prodicate-demo
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: http://www.taobao.com
        predicates:
          - Path=/between
          - name: Between
            args:
              datetime1: 2018-06-16T11:50:10.785+08:00[Asia/Shanghai]
              datetime2: 2020-06-16T11:50:10.785+08:00[Asia/Shanghai]
```


如果这个时候去访问是可以访问taobao网的,如果时间修改在2020到2022年的话就不能访问的到。


## Cookie路由断言工厂

Cookie路由断言工厂会取两个参数--cookie名称所对应的key和value.当请求中携带的cookie和Cookie断言工厂中配置的cookie一致的话,则路由匹配成功,否则就匹配不成功。


```yml
 - id: cookie_route
        uri: http://localhost:8084
        predicates:
          - Path=/test
          - Cookie=name,wuxiaobo
```


## Header路由断言工厂

Header路由断言工厂用于根据配置的路由Header信息进行断言匹配路由,匹配成功则进行转发,否则不进行转发。示例参照如下的配置:

```yml
spring:
  application:
    name: gateway-prodicate-demo
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: http://localhost:8084
        predicates:
          - Path=/hello
          - Header=X-Request-Id, \d+
      discovery:
        locator:
          enabled: true
```

## Host路由断言工厂

Host路由断言工厂是根绝配置的host,对请求中的Host进行断言处理,断言成功则进行路由转发,否则就不转发。配置如下:

```yml
spring:
  application:
    name: gateway-prodicate-demo
  cloud:
    gateway:
      routes:
       - id: host_route
        uri: https://jd.com
        predicates:
          - Path=/host
          - Host=**.baidu.com:8080
      discovery:
        locator:
          enabled: true
```

##  Method路由断言工厂

Method路由断言工厂会根据路由信息配置的method对请求方法是Get或者是Post方法等进行断言匹配，如果匹配成功则进行转发,否则就处理失败


```yml
spring:
  application:
    name: gateway-prodicate-demo
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://jd.com
        predicates:
          - Path=/method
          - Method=Get
      discovery:
        locator:
          enabled: true
```

然后访问 http://localhost:8138/method这个路径就可以跳转到京东的首页啦。

## Query路由断言工厂

Query路由断言工厂会从请求中获取两个参数,将请求中的参数和Query断言路由中的配置进行匹配,如果配置成功则转发,否则就转发失败。


```yml
spring:
  application:
    name: gateway-prodicate-demo
  cloud:
    gateway:
      routes:
       - id: query_route
         uri: https://baidu.com
         predicates:
          - Path=/query
          - Query=name, wuxiaobo
      discovery:
        locator:
          enabled: true
```

然后通过访问 http://localhost:8138/query?name=wuxiaobo这个路径就可以去访问百度了。



## RemoteAddr路由断言工厂

RemoteAddr路由断言工厂配置一个IPv4或者IPv6网段的字符串或者是IP，当请求的IP地址在网段之内或者是和配置的IP相同,则表示匹配成功,然后进行转发,否则就不能转发。

```yml
spring:
  application:
    name: gateway-prodicate-demo
  cloud:
    gateway:
      routes:
       - id: addr_route
        uri: https://baidu.com
        predicates:
          - Path=/addr
          - RemoteAddr=0:0:0:0:0:0:0:1
        locator:
          enabled: true
```


然后通过访问 http://localhost:8138/addr 这个地址可以访问百度。这里注意一下,因为我电脑使用的IPv6的地址,，所以配置的是0:0:0:0:0:0:0:1。



