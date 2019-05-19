# Zuul饥饿加载

```yml
zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-gray-zuul-client
#开启zuul的饥饿加载
  ribbon:
    eager-load:
      enabled: false
```

## 请求体修改

```java

public class ModifyRequestEntityFilter extends ZuulFilter {

	@Override
	public String filterType() {
		return PRE_TYPE;
	}

	@Override
	public int filterOrder() {
		return PRE_DECORATION_FILTER_ORDER + 1;// =6
	}

	@Override
	public boolean shouldFilter() {
		return true;
	}

	@Override
	public Object run() throws ZuulException {
		RequestContext ctx = RequestContext.getCurrentContext();
		HttpServletRequest request = ctx.getRequest();
		request.getParameterMap();
		Map<String, List<String>> requestQueryParams = ctx.getRequestQueryParams();
		if (requestQueryParams == null){
			requestQueryParams = new HashMap<>();
		}
		//这里添加新增参数的value，注意，只取list的0位
		ArrayList<String> arrayList = new ArrayList<>();
		arrayList.add("1wwww");
		requestQueryParams.put("test", arrayList);
		ctx.setRequestQueryParams(requestQueryParams);
		return null;
	}
}

```
>&nbsp;在网上找到一篇比较好的博客。关于使用Zuul对参数进行加密的可以看一下哦。https://www.cnblogs.com/yuki67/p/9561525.html

# 使用OkHttp

```xml
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>3.11.0</version>
        </dependency>
```

```yml
ribbon:
  okhttp:
    enabled: true
  http:
    client:
      enabled: false
```

# 开启重试机制


```xml

        <dependency>
            <groupId>org.springframework.retry</groupId>
            <artifactId>spring-retry</artifactId>
            <version>1.2.2.RELEASE</version>
        </dependency>
```

```yml
zuul:
  routes:
    client-a:
      path: /client/**
      serviceId: zuul-eureka-gray-zuul-client
  retryable: true
ribbon:
  #重试机制配置
  ConnectTimeout: 3000
  ReadTimeout: 60000
  MaxAutoRetries: 1 #对第一次请求的服务的重试次数
  MaxAutoRetriesNextServer: 1 #要重试的下一个服务的最大数量（不包括第一个服务）
  OkToRetryOnAllOperations: true
```
# Header传递


```java
public class HeaderDeliverFilter extends ZuulFilter {

	@Override
	public String filterType() {
		return PRE_TYPE;
	}

	@Override
	public int filterOrder() {
		return PRE_DECORATION_FILTER_ORDER + 1;// =6
	}

	@Override
	public boolean shouldFilter() {
		return true;
	}

	@Override
	public Object run() throws ZuulException {
		RequestContext context = RequestContext.getCurrentContext();
		context.addZuulRequestHeader("result", "to next service");
		return null;
	}
}
```

# 集合Swagger

```xml
        <dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>2.7.0</version>
		</dependency>
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>2.7.0</version>
		</dependency>
```

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

	@Autowired
	ZuulProperties properties;

	@Primary
	@Bean
	public SwaggerResourcesProvider swaggerResourcesProvider() {
		return () -> {
			List<SwaggerResource> resources = new ArrayList<>();
			properties.getRoutes().values().stream()
					.forEach(route -> resources.add(createResource(route.getServiceId(), route.getServiceId(), "2.0")));
			return resources;
		};
	}

	private SwaggerResource createResource(String name, String location, String version) {
		SwaggerResource swaggerResource = new SwaggerResource();
		swaggerResource.setName(name);
		swaggerResource.setLocation("/" + location + "/v2/api-docs");
		swaggerResource.setSwaggerVersion(version);
		return swaggerResource;
	}

}
```