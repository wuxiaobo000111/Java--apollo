
# 补充

关于这个知识点的代码地址如下所示:https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/gateway-group/gateway-service 和
https://github.com/wuxiaobo000111/java-framework/tree/master/spring-cloud-group/gateway-group/gateway-inner-filter-demo。 每个地方的代码都只是会指定除和这一部分有关的代码。

# 概述

&nbsp;&nbsp;&nbsp;&nbsp;Spring Cloud Gatewat中内置很多的路由过滤工厂,当然可以自己根据实际应用场景需要定制的自己的路由过滤器工厂。路由过滤器允许以某种方式修改请求进来的http请求或返回的http响应。路由过滤器主要作用于需要处理的特定路由。Spring Cloud Gateway提供了很多种类的过滤器工厂,过滤器的实现类将近二十多个。总得来说,可以分为七类: Header,Parameter.Path、 Status, Redirect跳转、Hytrix熔断和RateLimiter,下面介绍一下Spring Cloud Gateway中的Filter工厂。


# AddRequestHeader过滤器工厂

AddRequestHeader过滤器工厂的作用就是对于匹配的上的请求加上Header。

```yml
spring:
  application:
    name: gateway-inner-filter-demo
  cloud:
      gateway:
        routes:
        - id: add_request_parameter_route
          uri: http://localhost:8140
          filters:
          - AddRequestHeader=X-Request-Acme, ValueB
          # 不能少
          predicates:
          - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

```java

	/**
	 * @param request
	 * @param response
	 * @return
	 */
	@GetMapping("/test/head")
	public String testGatewayHead(HttpServletRequest request, HttpServletResponse response){
		String head=request.getHeader("X-Request-Acme");
		return "return head info:"+head;
	}

```

通过访问 http://localhost:8139/test/head  可以拿到想要的结果。


# AddRequestParameter过滤器

该过滤器作用是对匹配上的请求路由添加请求参数。

```yml

spring:
  application:
    name: gateway-inner-filter-demo
  cloud:
      gateway:
        routes:
#        - id: add_request_head_route
#          uri: http://localhost:8140
#          filters:
#          - AddRequestHeader=X-Request-Acme, ValueB
#          # 不能少
#          predicates:
#          - After=2017-01-20T17:42:47.789-07:00[America/Denver]
         - id: add_request_parameter_route
           uri: http://localhost:8140
           filters:
           - AddRequestParameter=example, wuxiaobo
           # 不能少
           predicates:
           - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

```java

	@GetMapping("/test/addRequestParameter")
	public String addRequestParameter(HttpServletRequest request, HttpServletResponse response){
		String parameter=request.getParameter("example");
		return "return addRequestParameter info:"+parameter;
	}


```

然后去请求 http://localhost:8139/test/addRequestParameter  这个地址就行了。



其他的功能就不在这里一一列举了,有兴趣的小伙伴可以访问 https://cloud.spring.io/spring-cloud-gateway/single/spring-cloud-gateway.html#_preservehostheader_gatewayfilter_factory 官方网站进行查询。


