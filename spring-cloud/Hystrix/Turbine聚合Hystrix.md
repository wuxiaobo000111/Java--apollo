
# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;Turbine就是聚合所有相关的Hystrix.stream流的方案,然后在Hystrix DashBoard中显示。在这个项目我们需要借助于原来的三个项目。具体项目地址参考[hystrix-eureka-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/hystrix-group/hystrix-eureka-demo  "hystrix-eureka-demo")、[feign-eureka-hystrix-producer](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/feign-group/feign-eureka-hystrix-producer "feign-eureka-hystrix-producer")、
[feign-eureka-hystrix-consumer](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/feign-group/feign-eureka-hystrix-consumer  "feign-eureka-hystrix-consumer")、[hystrix-eureka-turbine ](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/hystrix-group/hystrix-eureka-turbine  "hystrix-eureka-turbine")。下面讲述一下具体的代码实现。


# 项目启动

>&nbsp;&nbsp;&nbsp;&nbsp;首先启动hystrix-eureka-demo、feign-eureka-hystrix-producer、
feign-eureka-hystrix-consumer这三个项目。

# hystrix-eureka-turbine

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
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
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
package com.bobo.springcloud.learn.hystrixeurekaturbine;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
import org.springframework.cloud.netflix.turbine.EnableTurbine;

@SpringBootApplication
@EnableHystrix
@EnableHystrixDashboard
@EnableDiscoveryClient
@EnableTurbine
public class HystrixEurekaTurbineApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixEurekaTurbineApplication.class, args);
    }

}

```
## 配置

```yml
server:
  port: 8094
spring:
  application:
    name: hystrix-eureka-turbine
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
management:
  endpoints:
    web:
      exposure:
        include: "*"
turbine:
  app-config: hystrix-eureka-demo,feign-eureka-hystrix-consumer
  cluster-name-expression: "'default'"
```

# 结果


>&nbsp;&nbsp;&nbsp;&nbsp;首先启动hystrix-eureka-turbine这个项目。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/10.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;在地址栏中输入http://localhost:8094/hystrix这个地址,是进入turbine。然后在输入框中输入http://localhost:8094/turbine.stream。进入之后就可以看到两个项目的监控


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/11.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/12.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;注意,这里是个懒加载,必须在访问接口之后才会出现结果。
