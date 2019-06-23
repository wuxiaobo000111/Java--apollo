# 介绍

&nbsp;&nbsp;&nbsp;&nbsp;在前面介绍Zuul的章节中,我们知道Spring Cloud对Zuul进行封装处理之后;当通过Zuul访问后端微服务时,基于服务发现的默认路由规则是:http://zul-host:zul port/微服务在Eureka上的serviceld/**, Spring Cloud Gateway在设计的时候考虑从Zuul迁移到Gateway的兼容性和迁移成本等, Gateway基于服务发现的路由规则和Zuul的设计类似,但是也有很大差别。但是Spring Cloud Gateway基于服务发现的路由规则,在不同注册中心下其差异如下:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果把Gateway注册到Eureka上,通过网关转发服务调用,访问网关的URL是
	http://Gateway_HOST:Gateway-PORT/大写的serviceld/*,其中服务名默认必须是大写,否则会抛404错误,如果服务名要用小写访问,可以在属性配置文件里面加
spring.cloud.gateway.discovery.locator.lowerCaseServiceld-true配置解决。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果把Gateway注册到Zookeeper上,通过网关转发服务调用,服务名默认小写,因此不需要做任何处理。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果把Gateway注册到Consul上,通过网关转发服务调用,服务名默认小写,也不需要做人为修改。

&nbsp;&nbsp;&nbsp;&nbsp;下面会给出示例代码。

# 示例


https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/gateway-group/gateway-eureka-gateway

https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/gateway-group/gateway-eureka-producer

https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/gateway-group/geteway-eureka-consumer

这里给出了项目地址。

