>一部分信息来自于《重新定义Spring Cloud实战一书》。这是一本特别好的书。希望大家有时间了可以阅读一波
# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;consul-discovery在功能上分成为服务注册和服务发现。


# 服务注册

>&nbsp;&nbsp;&nbsp;&nbsp;服务启动时,会通过ConsulServiceRegistry.register()向Consul注册自身的服务。注册时,会告诉Consul以下信息。

```text
1.ID,服务ID,默认是服务名-端口号。
2.Name。服务名,默认是系统名称。
3.Tags。给服务打的标签,默认是[secure-false]。
4.Address。服务地址,默认是本机IP。
5.Port。服务端口,默认是服务的web端口。
6.Check.健康检查信息,包括Interval (健康检查间隔)和HTTP (健康检查地址)。
```

>&nbsp;&nbsp;&nbsp;&nbsp;一般情况下,我们不需要显式提供上述信息, spring-cloud-consul会有默认值。但是在一些特殊业务场景中,可能就需要定制上述服务了。consul discovery的常见配置如下所示



![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/42.jpg?raw=true)

## 如何自定义注册信息

### 依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-all</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```


### 配置文件

```yml
spring:
  application:
    name: consul-register
  cloud:
    consul:
      host: 127.0.0.1    # consul 启动地址
      port: 8500         # consul 启动端口
      discovery:
        prefer-ip-address: true     # 优先使用 IP 注册
        ip-address: 127.0.0.1       # 假装部署在 docker 中,指定了宿主机 IP
        port: 8080                  # 假装部署在 docker 中,指定了宿主机端口
        health-check-interval: 20s  # 健康检查间隔时间为 20s
        health-check-path: /health  # 自定义健康检查路径
        tags: ${LANG},test          # 指定服务的标签, 用逗号隔开

```

### 启动类

```java
package cn.springcloud.book.consul.register;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * consul-server 的启动主类
 * 为了简化代码，我们将 Controller 代码放在主类中，实际工作中不建议这么做
 */
@RestController
@SpringBootApplication
public class ConsulRegisterApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsulRegisterApplication.class,args);
    }
    /**
     * 这里我们不使用默认的健康检测, 而是使用自己定义的接口
     * @return SUCCESS
     */
    @GetMapping("/health")
    public String health(){
        return "SUCCESS";
    }
}

```

# 服务发现的实现

>&nbsp;&nbsp;&nbsp;&nbsp;在Consul中可以通过三种方式来调用服务:Ribbon、Feign和DiscoveryClient。这里涉及到的项目有五个,[consule-discovery-server-tag1](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/consul-group/consule-discovery-server-tag1 "consule-discovery-server-tag1")、[consule-discovery-server-tag2](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/consul-group/consule-discovery-server-tag2 "consule-discovery-server-tag2")、[consul-discovery-client-ribbon](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/consul-group/consul-discovery-client-ribbon "consul-discovery-client-ribbon")、[consul-discovery-client-feign](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/consul-group/consul-discovery-client-feign "consul-discovery-client-feign")、[consul-discovery-client-discoveryclient ](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/consul-group/consul-discovery-client-discoveryclient "consul-discovery-client-discoveryclient ")。首先让我们先来准备服务方。

## 两个服务

>&nbsp;&nbsp;&nbsp;&nbsp;其中consule-discovery-server-tag2只是和consule-discovery-server-tag1的tag不一样,其他内容都是一样的。

### pom依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
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
spring:
  application:
    name: consule-discovery-server-tag
  cloud:
    consul:
      host: 127.0.0.1
      port: 8500
      discovery:
#      如果是多个,请用逗号隔开
        tags: tag1
server:
  port: 8113

```

### 启动类

```java
package com.bobo.springcloud.learn.consulediscoveryservertag1;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConsuleDiscoveryServerTag1Application {

    public static void main(String[] args) {
        SpringApplication.run(ConsuleDiscoveryServerTag1Application.class, args);
    }

}

```

### 接口

```java
package com.bobo.springcloud.learn.consulediscoveryservertag1.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {


    @GetMapping(value = "/hello")
    public String hello (@RequestParam String name) {
        return name+" hello";
    }
}

```


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/55.jpg?raw=true)



>&nbsp;&nbsp;&nbsp;&nbsp;然后启动两个项目


##  consul-discovery-client-ribbon

### 依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
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
  port: 8115             # 因为本地启动，防止端口冲突
spring:
  application:
    name: consul-discovery-client-ribbon
  cloud:
    consul:
      host: 127.0.0.1    # consul 启动地址
      port: 8500         # consul 启动端口
      discovery:
        server-list-query-tags:     # 注意 server-list-query-tags 是一个 map
          consul-provider: tag1     # 在调用consul-provider 服务时, 使用 tag1 对应的实例
```

### 启动类

```java

package com.bobo.springcloud.learn.consuldiscoveryclientribbon;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class ConsulDiscoveryClientRibbonApplication {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate () {
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(ConsulDiscoveryClientRibbonApplication.class, args);
    }

}

```

### 接口

```java
package com.bobo.springcloud.learn.consuldiscoveryclientribbon.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class HelloController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping(value = "/hello")
    public String hello(@RequestParam String name) {
        String object = restTemplate.getForObject("http://consule-discovery-server-tag/hello?name=" + name, String.class);
        return object;
    }
}


```

## consul-discovery-client-feign

### 依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
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
  port: 8116             # 因为本地启动，防止端口冲突
spring:
  application:
    name: consul-discovery-client-feign
  cloud:
    consul:
      host: 127.0.0.1    # consul 启动地址
      port: 8500         # consul 启动端口
      discovery:
        server-list-query-tags:     # 注意 server-list-query-tags 是一个 map
          consul-provider: tag1     # 在调用consul-provider 服务时, 使用 tag1 对应的实例
```

### 启动类

```java  
package com.bobo.springcloud.learn.consuldiscoveryclientfeign;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class ConsulDiscoveryClientFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulDiscoveryClientFeignApplication.class, args);
    }

}

```

### Feign

```java
package com.bobo.springcloud.learn.consuldiscoveryclientfeign.feign;

import feign.Logger;
import feign.Request;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {

    @Bean
    public Request.Options options() {
        return new Request.Options(3000, 3000);
    }


    /**
     *
     * Logger.Level 的具体级别如下：
     NONE：不记录任何信息
     BASIC：仅记录请求方法、URL以及响应状态码和执行时间
     HEADERS：除了记录 BASIC级别的信息外，还会记录请求和响应的头信息
     FULL：记录所有请求与响应的明细，包括头信息、请求体、元数据
     * @return
     */
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
package com.bobo.springcloud.learn.consuldiscoveryclientfeign.feign;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(value = "consule-discovery-server-tag",configuration = FeignConfig.class)
public interface HelloService {

    @GetMapping(value = "/hello")
    String hello(@RequestParam(value = "name") String name);

}



```

### 接口

```java
package com.bobo.springcloud.learn.consuldiscoveryclientfeign.controller;

import com.bobo.springcloud.learn.consuldiscoveryclientfeign.feign.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    private HelloService helloService;

    @GetMapping(value = "/hello")
    public String hello(@RequestParam String name) {
        return helloService.hello(name);
    }
}

```

## consul-discovery-client-discoveryclient

### 依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
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
  port: 8118             # 因为本地启动，防止端口冲突
spring:
  application:
    name: consul-discovery-client-discoveryclient
  cloud:
    consul:
      host: 127.0.0.1    # consul 启动地址
      port: 8500         # consul 启动端口
      discovery:
        server-list-query-tags:     # 注意 server-list-query-tags 是一个 map
          consul-provider: tag1     # 在调用consul-provider 服务时, 使用 tag1 对应的实例
```

### 启动类

```java
package com.bobo.springcloud.learn.consuldiscoveryclientdiscoveryclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConsulDiscoveryClientDiscoveryclientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulDiscoveryClientDiscoveryclientApplication.class, args);
    }

}

```

### 接口

```java
package com.bobo.springcloud.learn.consuldiscoveryclientdiscoveryclient.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cglib.proxy.CallbackHelper;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.lang.annotation.Repeatable;
import java.util.List;

@RestController
public class HelloController {
    @Autowired  // ConsulDiscoveryClient 会在程序启动时,初始化为DiscoveryClient实例
    private DiscoveryClient discoveryClient;

    // 这里只举例获取服务方信息,不去请求服务方接口
    @GetMapping("/getServer")
    public List<ServiceInstance> getServer(String serviceId){
        return discoveryClient.getInstances(serviceId);
    }


    @GetMapping("/hello")
    public String hello(@RequestParam String name) {
        List<ServiceInstance> instances = discoveryClient.getInstances("consule-discovery-server-tag");
        RestTemplate restTemplate = new RestTemplate();
        if (instances != null && instances.size() > 0) {
            String serviceUri = instances.get(0).getUri().toString()+"/hello?name=" + name;
            ResponseEntity< String > restExchange =
                    restTemplate.exchange(
                            serviceUri,
                            HttpMethod.GET,
                            null, String.class);
            return restExchange.getBody();
        }
        return null;
    }

}

```

# 服务发现


>&nbsp;&nbsp;&nbsp;&nbsp;Consul Config刷新原理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们知道, Spring Cloud Consul是通过HTTP的方式跟Consul交互,那配置是如何实时生效的呢?答案其实很简单,那就是配置并没有实时生效。org.springframework.cloud.consul.config.ConfigWatch中有一个定时方法watchConfigKeyValues(),它默认每秒执行一次(可以通过spring.cloud.consul.config.watch.delay自定义),去Consul中获取最新的配置信息,一旦配置发生改变, Spring通过ApplicationEventPublisher重新刷新配置。Consul Config组件就是通过这种方式,达到配置“实时生效”的目的。那客户端如何得知配置被更新过了呢,答案在Consul返回的数据里。Consul会给每一项配置加一个"consullndex"属性,类似于版本号,如果配置更新,它就会自增。Spring Cloud Consul Config就是通过缓存"consullndex"来判断配置是否发生改变。
&nbsp;&nbsp;&nbsp;&nbsp;Consul Config高级配置
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Consul只支持用K/V的方式进行配置,那怎么让Consul支持同时配置多条的方式呢?难道要给Consul增加一个导入功能吗?聪明的读者马上就能想到, K/V不仅可以代表一条配置,还可以代表一个应用的配置,我们将应用名作为K, V中用来存放它所有的配置,这样就可以达到同时配置多条的效果。代码如下所示:


## 依赖

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
# application.yml

server:
  port: 8119
spring:
  profiles:
    active: test    # 指定启动时的 profiles

# bootstarp.yml

spring:
  application:
    name: consule-config-customize
  cloud:
    consul:
      config:
        format: yaml              # Consul 中 Value 配置格式为 yaml
        prefix: configuration     # Consul 中配置文件目录为 configuration, 默认为 config
        default-context: app      # 去该目录下查找缺省配置,默认为 application
        profile-separator: ':'    # profiles配置分隔符,默认为‘,’
        data-key: data
      host: localhost
      port: 8500          # 如果指定配置格式为 yaml 或者 properties, 则需要该值作为key,默认为 data

```

## 启动类

```java
package com.bobo.springcloud.learn.consulconfigcustomize;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConsulConfigCustomizeApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulConfigCustomizeApplication.class, args);
    }

}

```

## 配置

>&nbsp;&nbsp;&nbsp;&nbsp;配置的路径是/configuration/onsule-config-customize:test/data。这样的配置可以指定的是一个服务的配置文件
```yml
foo:
  bar:
    name: wuxiaobo111
server:
  port: 8119
```
