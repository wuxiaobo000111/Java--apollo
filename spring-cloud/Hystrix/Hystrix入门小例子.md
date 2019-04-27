
# 创建项目

>具体项目地址参考[hystrix-eureka-demo](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/hystrix-group/hystrix-eureka-demo  "eureka-client")。下面讲述一下具体的代码实现。

# pom依赖

```xml

<dependencies>
        <!--web依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--eureka client依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--hystrix依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!--hystrix 面板依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <!--监控依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--测试依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

```

# 启动类

```java
package com.bobo.springcloud.learn.hystrixeurekademo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
@EnableHystrixDashboard
@EnableCircuitBreaker
public class HystrixEurekaDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixEurekaDemoApplication.class, args);
    }

}

```

>&nbsp;&nbsp;&nbsp;&nbsp;@EnableDiscoveryClient表示是eureka 的一个客户端程序。@EnableHystrix表示是否开启Hystrix。@EnableHystrixDashboard表示是否开启Hystrix的面板。@EnableCircuitBreaker表示是启动断路器。

# 配置文件

```yml
server:
  port: 8091
spring:
  application:
    name: hystrix-eureka-demo
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

# 接口文件

```java
package cn.springcloud.book.service;

public interface IUserService {
    public String getUser(String username) throws Exception;

}

package com.bobo.springcloud.learn.hystrixeurekademo.service.impl;


import com.bobo.springcloud.learn.hystrixeurekademo.service.IUserService;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.stereotype.Component;

/**
 */
@Component
public class UserService implements IUserService {
	
	@Override
	@HystrixCommand(fallbackMethod="defaultUser")
	public String getUser(String username) throws Exception {
		if(username.equals("spring")) {
			return "This is real user";
		}else {
			throw new Exception();
		}
	}
	
	 /**
	  * 出错则调用该方法返回友好错误
	  * @param username
	  * @return
	  */
	 public String defaultUser(String username) {
	    return "The user does not exist in this system";
	 }
	 
}




package com.bobo.springcloud.learn.hystrixeurekademo.controller;
import com.bobo.springcloud.learn.hystrixeurekademo.service.IUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {

	@Autowired
	private IUserService userService;
	
	@GetMapping("/getUser")
	public String getUser(@RequestParam String username) throws Exception{
		return userService.getUser(username);
	}
}

```

# 结果验证

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/1.jpg?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/2.jpg?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/3.jpg?raw=true)


# Hystrix面板


>&nbsp;&nbsp;&nbsp;&nbsp;我们可以看到上述项目在启动的时候已经同时开启了Hystrix面板。这个面板如何使用的呢。当我们去访问http://localhost:8091/hystrix/这个路径的时候就可以进入到Hystrix的首页。可以看到Hystrix DashBoard支持三种不同的监控方式。


```text
默认的集群监控：通过 URL：http://turbine-hostname:port/turbine.stream 开启，实现对默认集群的监控。 

指定的集群监控：通过 URL：http://turbine-hostname:port/turbine.stream?cluster=[clusterName] 开启，
实现对 clusterName 集群的监控。 

单体应用的监控： 通过 URL：http://hystrix-app:port/hystrix.stream 开启 ，实现对具体某个服务实例的监控。
（现在这里的 URL 应该为 http://hystrix-app:port/actuator/hystrix.stream，Actuator 2.x 以后endpoints 全
部在/actuator下，可以通过management.endpoints.web.base-path修改）。

Delay：控制服务器上轮询监控信息的延迟时间，默认为 2000 毫秒，可以通过配置该属性来降低客户端的网络和 CPU 消耗。


Title：该参数可以展示合适的标题。


前两者都对集群的监控，需要整合 Turbine 才能实现。
```


>&nbsp;&nbsp;&nbsp;&nbsp;我们在yml文件中配置了management.endpoints.web.exposure.include,management.endpoints.web.exposure.include这个是用来暴露 endpoints 的。由于 endpoints 中会包含很多敏感信息，除了 health 和 info 两个支持 web 访问外，其他的默认不支持 web 访问。


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/4.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/5.jpg?raw=true)


## 结果

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/6.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;对于其中的面板中的参数如下图所示。
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-27/7.png?raw=true)
