
# 概述 

>&nbsp;&nbsp;&nbsp;&nbsp;本篇文章主要介绍的如何实现eureka服务注册的功能。这里分成了两个子项目,具体项目地址参考[eureka-client](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/eureka-client "eureka-client")和[eureka-server](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/eureka-server "eureka-server")。下面讲述一下eureka的代码。

# eureka-server

## 引入pom文件

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
    <groupId>com.bobo.springcloud</groupId>
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-server</name>
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
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
```java
package com.bobo.springcloud.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
//这个注解就表示是eureka server端
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}

```


## 配置文件
```yaml
# 服务端口
server:
  port: 8761
# eureka访问地址  
eureka:
  instance:
    hostname: localhost
  client:
# eureka服务端不向自己注册,因为是服务端,所以
    register-with-eureka: false
    fetch-registry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
  server:
    wait-time-in-ms-when-sync-empty: 0
    enable-self-preservation: false
```

## 结果


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-16/1.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;<font color=red>THE SELF PRESERVATION MODE IS TURNED OFF. THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS. 我们可以看到在上图中限制这个问题。原因如下所示:
```yaml
client:
#   eureka服务端不向自己注册
    register-with-eureka: false
#   是否允许客户端向Eureka 注册表获取信息，一般服务器为设置为false,客户端设置为true
    fetch-registry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
  server:
    wait-time-in-ms-when-sync-empty: 0
     # 是否开启自我保护机制
        ## 在分布式系统设计里头，通常需要对应用实例的存活进行健康检查，这里比较关键的问题就是要处
        #理好网络偶尔抖动或短暂不可用时造成的误判。另外Eureka Server端与Client端之间如果出现网
        #络分区问题，在极端情况下可能会使得Eureka Server清空部分服务的实例列表，这个将严重影响到
        #Eureka server的 availibility属性。因此Eureka server引入了SELF PRESERVATION机制。
        ## Eureka client端与Server端之间有个租约，Client要定时发送心跳来维持这个租约，表示自
        #己还存活着。 Eureka通过当前注册的实例数，去计算每分钟应该从应用实例接收到的心跳数，如果最
        #近一分钟接收到的续约的次数小于指定阈值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息。
        # 此处关闭可以防止问题（测试环境可以设置为false）：Eureka server由于开启并引入了SELF
        #PRESERVATION模式，导致registry的信息不会因为过期而被剔除掉，直到退出SELF PRESERVATION模式才能剔除。
    enable-self-preservation: false
```


>&nbsp;&nbsp;&nbsp;&nbsp;<font color=red>EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY’RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE. 还有可能出现这种问题。解决方案如下:
```text
、在生产上可以开自注册，部署两个server 
2、在本机器上测试的时候，可以把比值调低，比如0.49 
3、简单粗暴把自我保护模式关闭
```

# eureka-client

## 引入pom文件

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
    <groupId>com.bobo.springcloud</groupId>
    <artifactId>eureka-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-client</name>
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

## 启动类
```java
package com.bobo.springcloud.eurekaclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
/**
 * eureka客户端
 */
@EnableDiscoveryClient
public class EurekaClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }

}

```


## 配置文件
```yaml
server:
  port: 8082
spring:
  application:
    name: eureka-client
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

```

## 结果

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-16/1.jpg?raw=true)