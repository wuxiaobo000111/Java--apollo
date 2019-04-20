# 概述
>&nbsp;&nbsp;&nbsp;&nbsp;为了更好的模拟企业中开发。现在采用集群的模式启动。同样是针对于Feign入门教程中的例子。首先先启动Eureka集群模式。由于本地机器的限制,这里只是演示一下两台机器的集群。

# eureka集群模式。

>&nbsp;&nbsp;&nbsp;&nbsp;我这里采用的是虚拟机的方式启动的eureka集群。首先说一下eureka集群模式的原理:集群模式就是多个eureka服务相互注册。集群复制又两种复制方式,一种是主从复制(比较常见的就是数据库的主从和zookeeper的主从),相信大家都很熟悉了。还有一种就是对等复制(Peer to Peer模式),副本之间不分主从,任何副本都可以接受写操作,然后每个副本之间进行相互更新。

# eureka单机配置:
```yml
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
#   eureka服务端不向自己注册
    register-with-eureka: false
#   是否允许客户端向Eureka 注册表获取信息，一般服务器为设置为false,客户端设置为true
    fetch-registry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
  server:
    wait-time-in-ms-when-sync-empty: 0
```

# 集群配置

>&nbsp;&nbsp;&nbsp;&nbsp;需要两个eureka实例,所以指定的端口并不一样。端口为8761的向8760的实例上注册,端口为8760的向8761上进行注册。这样通过相互注册的形式就可以形成eureka集群。

```yml
#注册中心运行的端口号
server:
  port: 8760
#注册中心应用名称
spring:
  application:
      name: eureka-server
#eureka.server.enableSelfPreservation:是否向注册中心注册自己
#通过eureka.client.registerWithEureka：false和fetchRegistry：false来表明自己是一个eureka server.
eureka:
#  server:
#      enableSelfPreservation: false
  instance:
      hostname: peer2
      prefer-ip-address: false
#      ip-address: 172.193.225.185
#      instance-id: ${spring.cloud.client.ipAddress}:${server.port}
  client:
      service-url:
           defaultZone: http://peer1:8761/eureka/ 
```

```yml
#注册中心运行的端口号
server:
  port: 8761
#注册中心应用名称
spring:
  application:
      name: eureka-server
#eureka.server.enableSelfPreservation:是否向注册中心注册自己
#通过eureka.client.registerWithEureka：false和fetchRegistry：false来表明自己是一个eureka server.
eureka:
#  server:
#      enableSelfPreservation: false
  instance:
      hostname: peer1
      prefer-ip-address: false
#      ip-address: 172.193.225.185
#      instance-id: ${spring.cloud.client.ipAddress}:${server.port}
  client:
      service-url:
           defaultZone: http://peer1:8760/eureka/
```

>&nbsp;&nbsp;&nbsp;&nbsp;我这里是当更新配置文件的时候就打一个jar包,在很多博客上是根据profile进行区分的。然后上传到虚拟机上。这里其实是有很多坑的,最初的时候我是用IP用来做区分的,但是经常会出现一个eureka服务不可达。所以最后的解决方式就是在虚拟机中修改host文件,让hostname指定到对应的地址就行。


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/4.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;然后使用java命令分别启动两个jar包。启动命令是java -jar eureka-server-8760.jar --server.port=8760和java -jar eureka-server-8761.jar --server.port=8761启动。


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/6.jpg?raw=true)

# 修改入门教程中的两个项目

## feign-eureka-producer
```yml

feign-client-config:
  ConnectTimeout: 3000
  ReadTimeout: 3000
server:
  port: 8084
spring:
  application:
    name: feign-eureka-consumer
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
```

## feign-eureka-consumer
```yml

feign-client-config:
  ConnectTimeout: 3000
  ReadTimeout: 3000
server:
  port: 8084
spring:
  application:
    name: feign-eureka-consumer
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
```

# 验证Feign的集群能力

>&nbsp;&nbsp;&nbsp;&nbsp;这里可以启动两个feign-eureka-producer实例,指定不同的端口即可 。

## 实例1

```java
package com.bobo.springcloud.feign.feigneurekaproducer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author wuxiaobo
 */
@RestController
public class HelloController {

    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String hello(String name) {
        return name+" hello from producer1";
    }
}

```


## 实例二

```java
package com.bobo.springcloud.feign.feigneurekaproducer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author wuxiaobo
 */
@RestController
public class HelloController {

    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String hello(String name) {
        return name+" hello from producer2";
    }
}

```

# 结果

>&nbsp;&nbsp;&nbsp;&nbsp;启动feign-eureka-producer两个实例和feign-eureka-consumer一个实例,然后去eureka面板中查看一下结果:

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/6.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;然后我们去访问feign-eureka-consumer中的中的接口:

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/8.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/7.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;其实这个时候我们就会发现Feign在内部自动做了负载均衡。

# 验证eureka集群能力

>&nbsp;&nbsp;&nbsp;&nbsp;正如我们刚开始的时候是启动了两台eureka机器,现在我们shutdown一台机器,看一下结果。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/9.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/10.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;我shutdown掉的是端口为8760的机器,在eureka中显示是不可达的,但是当我们去访问feign的时候依然是可以访问到。
