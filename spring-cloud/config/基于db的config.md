# 概述

这里主要是介绍一下如何使用mysql作为config的配置中心的底层数据。这里给出了两个项目,两个项目的地址分别是[config-server-db-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/config-group/config-server-db-demo "config-server-db-demo")和[config-client-db-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/config-group/config-client-db-demo "config-client-db-demo")。

# 数据库文件

```mysql

-- 创建类型
CREATE TABLE `PROPERTIES` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `KEY` TEXT DEFAULT NULL,
  `VALUE` TEXT DEFAULT NULL,
  `APPLICATION` TEXT DEFAULT NULL,
  `PROFILE` TEXT DEFAULT NULL,
  `LABLE` TEXT DEFAULT NULL,
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;




INSERT INTO `spring-cloud-config`.`properties` (`ID`, `KEY`, `VALUE`, `APPLICATION`, `PROFILE`, `LABLE`) VALUES ('3', 'cn.springcloud.book.config', 'I am the mysql configuration file from dev environment.', 'config-info', 'dev', 'master');
INSERT INTO `spring-cloud-config`.`properties` (`ID`, `KEY`, `VALUE`, `APPLICATION`, `PROFILE`, `LABLE`) VALUES ('4', 'cn.springcloud.book.config', 'I am the mysql configuration file from test environment.', 'config-info', 'test', 'master');
INSERT INTO `spring-cloud-config`.`properties` (`ID`, `KEY`, `VALUE`, `APPLICATION`, `PROFILE`, `LABLE`) VALUES ('5', 'cn.springcloud.book.config', 'I am the mysql configuration file from prod environment.', 'config-info', 'prod', 'master');


```
# config-server-db-demo

## 依赖

```xml
  <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
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
  port: 8131
spring:
  application:
    name: config-server-db-demo
  cloud:
    config:
      server:
        jdbc:
          sql: SELECT `KEY`, `VALUE` FROM PROPERTIES WHERE application =? AND profile =? AND lable =?
      label: master
    refresh:
        refreshable: none
  profiles:
    active: jdbc

  ## 数据配置
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/spring-cloud-config?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
logging:
  level:
    org.springframework.jdbc.core: DEBUG
    org.springframework.jdbc.core.StatementCreatorUtils: Trace

```

## 启动类

```java
package com.bobo.springcloud.learn.configserverdbdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigServerDbDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerDbDemoApplication.class, args);
    }

}

```

# config-client-db-demo

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

## 配置

```yml
# application.yml
spring:
  application:
    name: config-client-db-demo

server:
  port: 8132

logging:
      level:
          root: INFO

# bootstrap.yml


spring:
    cloud:
        config:
            label: master
            uri: http://localhost:8131
            name: config-info
            profile: dev



```


## 启动类

```java
package com.bobo.springcloud.learn.configclientdbdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConfigClientDbDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientDbDemoApplication.class, args);
    }

}

```


## config

```java
package com.bobo.springcloud.learn.configclientdbdemo.config;

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

## controller

```java
package com.bobo.springcloud.learn.configclientdbdemo.controller;


import com.bobo.springcloud.learn.configclientdbdemo.config.ConfigInfoProperties;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RequestMapping("configConsumer")
public class ConfigClientController {

    @Autowired
    private ConfigInfoProperties configInfoValue;

    @RequestMapping("/getConfigInfo")
    public String getConfigInfo(){
        return configInfoValue.getConfig();
    }
}

```