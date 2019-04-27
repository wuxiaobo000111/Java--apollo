# Actuator的定义
>&nbsp;&nbsp;&nbsp;&nbsp;执行器是一个制造业术语，指的是用于移动或控制东西的一个机械装置，一个很小的改变就能让执行器产生大量的运动。 

# 开启Actuator

>&nbsp;&nbsp;&nbsp;&nbsp;spring-boot-actuator模块提供Spring Boot所有的production-ready特性，启用该特性的最简单方式是添加spring-boot-starter-actuator ‘Starter’依赖。 


```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

# 端点

>&nbsp;&nbsp;&nbsp;&nbsp;执行器端点（endpoints）可用于监控应用及与应用进行交互，Spring Boot包含很多内置的端点，你也可以添加自己的。例如，health端点提供了应用的基本健康信息。 每个端点都可以启用或禁用。这控制着端点是否被创建，并且它的bean是否存在于应用程序上下文中。要远程访问端点，还必须通过JMX或HTTP进行暴露,大部分应用选择HTTP，端点的ID映射到一个带/actuator前缀的URL。例如，health端点默认映射到/actuator/health。

>&nbsp;&nbsp;&nbsp;&nbsp;注意: Spring Boot 2.0的端点基础路径由“/”调整到”/actuator”下,如：/info调整为/actuator/info 
可以通过以下配置改为和旧版本一致
:management.endpoints.web.base-path=/


| ID|描述 | 默认启用|
|--|--|--|
|auditevents  | 显示当前应用程序的审计事件信息 |  Yes |
|beans	|显示一个应用中所有Spring Beans的完整列表|	Yes|
|conditions	|显示配置类和自动配置类(configuration and auto-configuration classes)的状态及它们被应用或未被应用的原因	|Yes|
|configprops|	显示一个所有@ConfigurationProperties的集合列表|	Yes|
|env|	显示来自Spring的 ConfigurableEnvironment的属性|	Yes|
|flyway	|显示数据库迁移路径，如果有的话	|Yes|
|health	|显示应用的健康信息（当使用一个未认证连接访问时显示一个简单的’status’，使用认证连接访问则显示全部信息详情）	|Yes|
|info	|显示任意的应用信息|	Yes|
|liquibase	|展示任何Liquibase数据库迁移路径，如果有的话	|Yes|
|metrics	|展示当前应用的metrics信息|	Yes|
|mappings	|显示一个所有@RequestMapping路径的集合列表|	Yes|
|scheduledtasks	|显示应用程序中的计划任务	|Yes|
|sessions	|允许从Spring会话支持的会话存储中检索和删除(retrieval and deletion)用户会话。使用Spring Session对反应性Web应用程序的支持时不可用。	|Yes|
|shutdown	|允许应用以优雅的方式关闭（默认情况下不启用）	|No|
|threaddump	|执行一个线程dump	|Yes|

>&nbsp;&nbsp;&nbsp;&nbsp;

| ID|描述 | 默认启用|
|--|--|--|
|heapdump	|返回一个GZip压缩的hprof堆dump文件	|Yes|
jolokia	|通过HTTP暴露JMX beans（当Jolokia在类路径上时，WebFlux不可用）	|Yes|
|logfile	|返回日志文件内容（如果设置了logging.file或logging.path属性的话），支持使用HTTP Range头接收日志文件内容的部分信息	|Yes|
|prometheus	|以可以被Prometheus服务器抓取的格式显示metrics信息	|Yes|

# 开启端点


>&nbsp;&nbsp;&nbsp;&nbsp;默认情况下，除shutdown以外的所有端点均已启用。要配置单个端点的启用，请使management.endpoint.<id>.enabled属性。以下示例启用shutdown端点：management.endpoint.shutdown.enabled=true。另外可以通过management.endpoints.enabled-by-default来修改全局端口默认配置,以下示例启用info端点并禁用所有其他端点：management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true

# 暴露端点

>&nbsp;&nbsp;&nbsp;&nbsp;下面是两种不同的服务器锁对应的默认的开启方式:

| ID|JMX | Web|
|--|--|--|
|auditevents|	Yes	|No|
|beans|	Yes	|No|
|conditions|	Yes	|No|
|configprops	|Yes|	No|
|env|	Yes	|No|
|flyway	|Yes	|No|
|health |Yes|	Yes|
|heapdump|	N/A|	No|
|httptrace	Y|es|	No|
|info|	Yes	|Yes|
|jolokia	|Yes|	No|
|logfile|	Yes|	Nov
|loggers	|Yes|	No|
|liquibase|	Yes|	No|
|metrics	|Yes|	No|
|mappings	|Yes|	No|
|prometheus	|N/A|	No|
|scheduledtasks|	Yes|	No|
|sessions	|Yes|	No|
|shutdown	|Yes|	No|
|threaddump	|Yes|	No|

>&nbsp;&nbsp;&nbsp;&nbsp;要更改公开哪些端点，请使用以下技术特定的include和exclude属性：

|  Property| Default |
|--|--|
|management.endpoints.jmx.exposure.exclude|	*|
|management.endpoints.jmx.exposure.include|	*|
|management.endpoints.web.exposure.exclude|	*|
|management.endpoints.web.exposure.include|	info, health|

>&nbsp;&nbsp;&nbsp;&nbsp;include属性列出了公开的端点的ID,exclude属性列出了不应该公开的端点的ID exclude属性优先于include属性。包含和排除属性都可以使用端点ID列表进行配置。这里的优先级是指同一端点ID,同时出现在include属性表和exclude属性表里,exclude属性优先于include属性,即此端点没有暴露

```text
注意 
*在YAML中有特殊的含义，所以如果你想包含（或排除）所有的端点，一定要加引号，如下例所示：
management:
  endpoints:
    web:
      exposure:
        include: '*'
如果您的应用程序对外公开，我们强烈建议您保护您的端点,方法见下文。 
如果您希望在暴露端点时实施您自己的策略，您可以注册一个EndpointFilter bean。
```

# 配置端点缓存的时间

>&nbsp;&nbsp;&nbsp;&nbsp;对于不带任何参数的读取操作,端点自动缓存对其相应。要配置端点缓存相应时间,以下示例将beans端点缓存的生存时间设置为10秒：management.endpoint.beans.cache.time-to-live=10s

# 端点的发现页

>&nbsp;&nbsp;&nbsp;&nbsp;“discovery page”添加了指向所有端点的链接。默认情况下，“discovery page”可通过/actuator访问。 需要注意的是,这里的/actuator指的是端点的基础路径,如果基础路径改变,发现页访问路径会跟着改变. 例如，如果基础路径是/manage，则发现页面可从/ manage获得 但是,当基础路径设置为/时，禁用发现页面以防止与其他映射发生冲突的可能性。 

# 端点的路径

>&nbsp;&nbsp;&nbsp;&nbsp;默认情况下，端点通过使用端点的ID在/actuator路径下的HTTP上公开。例如，beans端点暴露在/actuator/beans下。如果要将端点映射到其他路径，则可以使management.endpoints.web.path-mapping属性。另外，如果您想更改基本路径，则可以使用management.endpoints.web.base-path。 以下示例将/actuator/health重新映射到/healthcheck：

```properties
management.endpoints.web.base-path=/
management.endpoints.web.path-mapping.health=healthcheck
```

# 跨域支持


>&nbsp;&nbsp;&nbsp;&nbsp;跨源资源共享（Cross-origin resource sharing,CORS）是W3C规范，允许您以灵活的方式指定授权哪种跨域请求。如果您使用Spring MVC或Spring WebFlux，则可以配置Actuator的Web端点来支持这些场景。默认情况下，CORS支持处于禁用状态，只有在设置了management.endpoints.web.cors.allowed-origins属性后才能启用。以下配置允许来自example.com域的GET和POST调用：

```properties
management.endpoints.web.cors.allowed-origins=http://example.com
management.endpoints.web.cors.allowed-methods=GET,POST
```


# 实现自定义端点

>&nbsp;&nbsp;&nbsp;&nbsp;如果添加用@Endpoint注解的@Bean，则任何使用@ReadOperation，@WriteOperation或@DeleteOperation注释的方法都会自动通过JMX公开，并且也可以通过HTTP在Web应用程序中通过HTTP公开。也可以使用Jersey，Spring MVC或Spring WebFlux通过HTTP公开端点。

>&nbsp;&nbsp;&nbsp;&nbsp;您还可以使用@JmxEndpoint或@WebEndpoint编写技术特定的端点。这些端点仅限于各自的技术。例如，@WebEndpoint仅通过HTTP公开，而不通过JMX公开。

>&nbsp;&nbsp;&nbsp;&nbsp;您可以使用@EndpointWebExtension和@EndpointJmxExtension编写技术特定的扩展。这些注释可让您提供技术特定的操作，以增强现有端点。

>&nbsp;&nbsp;&nbsp;&nbsp;最后，如果您需要访问特定于Web框架的功能，则可以实现Servlet或Spring @Controller和@RestController端点，但代价是它们不能通过JMX或使用其他Web框架提供。


# //todo 不太理解


