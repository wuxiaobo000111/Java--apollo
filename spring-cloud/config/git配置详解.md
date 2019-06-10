# Git中URI占位符

Config Server支持占位符的使用,支持{application},{profile},{label},在配置URI的时候，可以通过应用名称来区分对应的仓库然后使用。两个项目的地址分别是[config-client-placeholders-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/config-group/config-client-placeholders-demo   "config-client-placeholders-demo")和[config-server-placeholders-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/config-group/config-server-placeholders-demo "config-server-placeholders-demo")。

# config-server-placeholders-demo
## pom依赖

```xml
      <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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
  cloud:
    config:
      server:
        git:
          uri: https://github.com/wuxiaobo000111/{application}
          username:
          password:
          search-paths: SC-BOOK-CONFIG
  application:
    name: config-server-placeholders-demo
server:
  port: 8127
logging:
  level:
    root: debug

```

在uri路径中使用的是占位符,和client中的spring.cloud.config.name对应

## 启动类

```java
package com.bobo.springcloud.learn.configserverdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigServerDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerDemoApplication.class, args);
    }

}


```



# config-server-placeholders-demo

和[config-client-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/config-group/config-client-demo "config-client-demo")项目基本上一致,需要改动的就是bootstrap.yml中的name属性,修改为spring-cloud-config。

配置中心的结构如下所示:

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/84.jpg?raw=true)
