# 概述

介绍一下spring cloud config中如何使用。这里给出了两个项目,两个项目的地址分别是[config-server-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/config-group/config-server-demo "config-server-demo")和[config-client-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/config-group/config-client-demo "config-client-demo")。

# config-server-demo

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
          uri: https://github.com/wuxiaobo000111/spring-cloud-config.git
          username: 
          password: 
          search-paths: SC-BOOK-CONFIG
  application:
    name: config-server-demo
server:
  port: 8125
```

其中uri对应的是git配置中心仓库的地址。username是github的登录账号,password是github的登录密码。search-paths表示的是哪一个文件夹。


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

@EnableConfigServer是说明这个是配置中心的服务端。

git项目中的目录结果如下所示:


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/82.jpg?raw=true)


启动项目,然后访问http://localhost:8125/config-info/test/master这个路径可以看到结果。这个路径其实有一定的讲究,官方给出的映射大致有如下几种情况:

```text
1. /{application}/{profile}[/{label}]

2./{appliaciton}-{profile}.yml

3. /{label}/{application}-{profile}.yml

4. /{application}-{profile}.properties

5. /{label}/{application}-{profile}.properties

application是应用名,也就是git文件名。如上所示是config-info,profile对应的是环境。label指的是
git的分支,如果不写则表示是master分支
```

# config-client-demo

## 依赖

```xml
  <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
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
# applicaton.yml

server:
  port: 8126

spring:
  application:
    name: config-client-demo
    
# bootstrap.yml

spring:
    cloud:
        config:
            label: master
            uri: http://localhost:8125
            name: config-info
            profile: dev

label对应的github仓库的分支
uri表示的是config server的地址
name 是对应的application
profile是指定的环境
```

## 启动类

```java
package com.bobo.springcloud.learn.configclientdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConfigClientDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientDemoApplication.class, args);
    }

}

```

## 配置

```java 
package com.bobo.springcloud.learn.configclientdemo.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "cn.springcloud.book")
public class ConfigInfoProperties {

    private String config;

    public String getConfig() {
        return config;
    }

    public void setConfig(String config) {
        this.config = config;
    }
}

```
## web接口

```java
package com.bobo.springcloud.learn.configclientdemo.controller;


import com.bobo.springcloud.learn.configclientdemo.config.ConfigInfoProperties;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class ConfigClientController {

    @Autowired
    private ConfigInfoProperties configInfoValue;

    @RequestMapping("/getConfigInfo")
    public String getConfigInfo(){
        return configInfoValue.getConfig();
    }
}

```



通过启动这两个项目就可以搭建一个config的简单小例子


# 如果刷新配置

## 手动刷新

首先,在客户端引入新的依赖

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

然后加入新的配置

```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always

```

## 添加注解

```java
@Component
@RefreshScope
@ConfigurationProperties(prefix = "cn.springcloud.book")
public class ConfigInfoProperties {

    private String config;

    public String getConfig() {
        return config;
    }

    public void setConfig(String config) {
        this.config = config;
    }
}

```

当需要修改配置的时候,就需要手动刷新配置,在客户端运行一个post请求：http://localhost:8126/actuator/refresh


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/83.jpg?raw=true)


这样就可以手动刷新了

## 通过bus刷新

这里我就不写了,因为国内现在又很多配置中心,使用最多的就是携程的[apollo](https://github.com/ctripcorp/apollo "config-server-demo"),很少使用bus刷新。所以这里也就不给出例子,有兴趣的可以自己搭建一个。之后会给出基于apollo和nacos的配置中心的例子
