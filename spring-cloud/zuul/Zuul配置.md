# 路由配置

>&nbsp;&nbsp;&nbsp;&nbsp;在示例代码中给出了如下的Zuul的配置。配置了从/client/**到zuul-eureka-client-demo的映射。

```yml

zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-client-demo
```

>&nbsp;&nbsp;&nbsp;&nbsp;可以进行如下简单的配置.也是可以实现功能。

```yml
zuul:
  routes:
    zuul-eureka-client-demo: /client/**
```

## 集群配置

>&nbsp;&nbsp;&nbsp;&nbsp;如果自己的服务是多个实例的话,是如何配置的嗯。现在我们修改一下zuul-eureka-client-demo的代码，然后启动两个实例。

### 实例一

### 接口文件

```java
package com.bobo.springcloud.learn.zuuleurekaclientdemo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping(value = "/hello")
    public String  hello (String name) {
        return "hello, "+ name+"I am from server1";
    }
}

```

### 配置文件

```yml
server:
  port: 8104
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
spring:
  application:
    name: zuul-eureka-client-demo


```

## 实例二

### 接口文件

```java
package com.bobo.springcloud.learn.zuuleurekaclientdemo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping(value = "/hello")
    public String  hello (String name) {
        return "hello, "+ name+"I am from server1";
    }
}


```

### 配置文件

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

### 修改zuul服务

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
ribbon:
  eureka:
    enabled: false
zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-client
zuul-eureka-client :
  ribbon:
    listOfServers: http://localhost:8103/,http://localhost:8104/
#zuul:
#  routes:
#    zuul-eureka-client-demo: /client/**
#zuul:
#  routes:
#    client-a:
#      path: /client/**
#      serviceId: zuul-eureka-client-demo
```


>&nbsp;&nbsp;&nbsp;&nbsp;在其他书上也有看到有如下配置的。

```yml

zuul:
  routes:
    ribbon-route:
      path: /ribbon/**
      serviceId: ribbon-route

ribbon:
  eureka:
    enabled: false  #禁止Ribbon使用Eureka

ribbon-route:
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule     #Ribbon LB Strategy
    listOfServers: localhost:7070,localhost:7071     #client services for Ribbon LB
```

>&nbsp;&nbsp;&nbsp;&nbsp;这样通过网关服务就能实现对一个服务的多实例进行网关管理。

## 路由通配符

| 规则 | 解释  |示例  |
|--|--|--|
| /** |  匹配任意数量的路径和字符 |  /client/add, /client/a/b/ccc|
| /* |  匹配任意数量的字符 |/client/a, /client/b,/client/ccc  |
| /? |  匹配单个字符 | /client/a, /client/b,/client/c |


# 功能配置

## 路由前缀

```yml
zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-client
  prefix: /bobo
```

>&nbsp;原来的路径http://127.0.0.1:8102/client/hello?name=wuxiaobo就变成了http://127.0.0.1:8102/bobo/client/hello?name=wuxiaobo。但是其实功能路径并没有发生改变。

## 服务屏蔽和路径屏蔽

```yml
zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-client
  prefix: /bobo
  ignored-services: client-b
  ignored-patterns: /**/danger/**
```

>&nbsp;&nbsp;&nbsp;&nbsp;ignored-services可以防止服务入侵。ignored-patterns表示是忽略的接口。上面加起来就是会忽略掉client-b服务的匹配到的/**/danger/**接口。

## 敏感头信息

```yml
zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-client
  prefix: /bobo
  sensitive-headers: Cookie.Set-Cookie
```

>&nbsp;&nbsp;&nbsp;&nbsp;sensitive-headers会忽略配置的头信息。不会传递给下游的服务。

## 重定向

```yml

zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-client
  prefix: /bobo
# 设置敏感头信息
  sensitive-headers: Cookie.Set-Cookie
# 解决重定向暴露服务地址的问题。
  add-host-header: true
```
## 重试

```yml
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-client
  prefix: /bobo
# 设置敏感头信息
  sensitive-headers: Cookie.Set-Cookie
# 解决重定向暴露服务地址的问题。
  add-host-header: true
  # 开启重试
  retryable: true
ribbon:
  MaxAutoRetries: 1 #同一个服务重试的次数(除去首次)
  # 切换相同服务数量
  MaxAutoRetriesNextServer: 1
```