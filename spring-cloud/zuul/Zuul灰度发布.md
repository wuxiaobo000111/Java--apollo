# 灰度发布

>&nbsp;&nbsp;&nbsp;&nbsp;在Eureka中存在两种元数据。

```text
    1. 标准元数据:存放的是服务的各种注册信息。比如IP、端口、服务健康状况。存储在专门为服务开辟的注册表中。

    2. 自定义元数据: 使用eureka.instance.metadata-map.<key>=value的方式来配置。在内部维持了一个Map来保存自定义的元数据信息。可以配置在远端服务,随着服务一并注册到Eureka注册表中。

    server:
    port: 8102
    eureka:
        client:
        serviceUrl:
             defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
        instance:
            metadata-map:
                name: bobo
```

# 代码

>&nbsp;&nbsp;&nbsp;&nbsp;参照[zuul-eureka-gray-zuul-client](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/zuul-group/zuul-eureka-gray-zuul-client "zuul-eureka-gray-zuul-client")和[zuul-eureka-gray-zuul-server](https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/zuul-group/zuul-eureka-gray-zuul-server "zuul-eureka-gray-zuul-server")两个项目的代码实现