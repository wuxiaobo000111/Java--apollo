# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这里主要给出关于基于consul作为配置中心的示例代码,这里给出了项目地址分别是[consul-demo-config](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/consul-group/consul-demo-config "consul-demo-config")

# 代码

## pom依赖

```xml

    <dependencies>
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
    </dependencies>
```

## 配置文件

```yml
spring:
  application:
    name: consul-demo-config
  cloud:
    consul:
      host: 127.0.0.1
      port: 8500
server:
  port: 8112
```

## 启动类

```java
package com.bobo.springcloud.learn.consuldemoconfig;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.context.config.annotation.RefreshScope;

@SpringBootApplication
@EnableDiscoveryClient
@RefreshScope
public class ConsulDemoConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulDemoConfigApplication.class, args);
    }

}


```

## 接口

```java
package com.bobo.springcloud.learn.consuldemoconfig;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    // 读取远程配置
    @Value("${name}")
    private String name;

    @GetMapping(value = "/getName")
    public String getName  () {
        return name;
    }

}

```