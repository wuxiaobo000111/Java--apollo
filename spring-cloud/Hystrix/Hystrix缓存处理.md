# Hystrix请求缓存

>&nbsp;&nbsp;&nbsp;&nbsp;Hystrix的缓存是在同一个请求中进行的,在进行第一次调用结束后对结果进行缓存。当有相同参数的请求过来的时候,会使用第一次请求的结果。
缓存的声明周期只有在这一次请求中有效。

>&nbsp;&nbsp;&nbsp;&nbsp;HystrixCommand使用缓存有两种方式,一种是继承,一种是使用注解的形式。注意第一次请求的结果要放在Hystrix上下文中,
所以最初的时候要实例化一个Hystrix上下文。这里采用了拦截器进行实现,代码如下所示：

```java
package com.bobo.springcloud.learn.hystrixeurekacache.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.mvc.Controller;

@Configuration
public class CacheConfiguration {
	
    @Bean
    @ConditionalOnClass(Controller.class)
    public CacheContextInterceptor userContextInterceptor() {
        return new CacheContextInterceptor();
    }

    @Configuration
    @ConditionalOnClass(Controller.class)
    public class WebMvcConfig extends WebMvcConfigurerAdapter {
        @Autowired
        CacheContextInterceptor userContextInterceptor;
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(userContextInterceptor);
        }
    }
}

package com.bobo.springcloud.learn.hystrixeurekacache.config;

import com.netflix.hystrix.strategy.concurrency.HystrixRequestContext;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CacheContextInterceptor implements HandlerInterceptor {
	
	private HystrixRequestContext context;

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
		// 初始化上下文
		context = HystrixRequestContext.initializeContext();
		return true;
	}

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {

	}

	//关闭上下文
	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
		context.shutdown();
	}
}



```
# 使用继承的方式实现缓存


>&nbsp;&nbsp;&nbsp;&nbsp;通过继承HystrixCommand,然后重写getCacheKey方法,用来保证对于同一个请求返回同样的键值。
对于清除缓存,则使用HystrixRequestCache类的clean方法
```java
package com.bobo.springcloud.learn.hystrixeurekacache.service;

import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandKey;
import com.netflix.hystrix.HystrixRequestCache;
import com.netflix.hystrix.strategy.concurrency.HystrixConcurrencyStrategyDefault;
import org.springframework.web.client.RestTemplate;

public class HelloCommand extends HystrixCommand<String>{

    private RestTemplate restTemplate;
    private Integer id;
    
    public HelloCommand(RestTemplate restTemplate, Integer id) {
        super(HystrixCommandGroupKey.Factory.asKey("springCloudCacheGroup"));
        this.id = id;
        this.restTemplate = restTemplate;
    }
    
    @Override
    protected String getFallback() {
        return "fallback";
    }

    @Override
    protected String run() {
		String json = restTemplate.getForObject("http://HYSTRIX-EUREKA-CACHE-PRODUCER/getUser/{1}", String.class, id);
		System.out.println(json);
		return json;
    }

    @Override
    protected String getCacheKey() {
        return String.valueOf(id);
    }
    
    public static void cleanCache(Long id){
        HystrixRequestCache.getInstance(HystrixCommandKey.Factory.asKey("springCloudCacheGroup"), HystrixConcurrencyStrategyDefault.getInstance()).clear(String.valueOf(id));
    }

}


/**
	 * 继承类方式
	 * @param id
	 * @return
	 */
	@RequestMapping(value = "/getUserIdByExtendCommand", method = RequestMethod.GET)
	public ResultMap getUserIdByExtendCommand(@RequestParam Integer id) {
		HelloCommand one = new HelloCommand(restTemplate,id);
		one.execute();
		System.out.println("from cache:   " + one.isResponseFromCache());
		HelloCommand two = new HelloCommand(restTemplate,id);
		two.execute();
		System.out.println("from cache:   " + two.isResponseFromCache());
	    return new ResultMap(0,"success");
	}
	
	
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/4.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;然后我们修改一下上述的接口,修改过后的接口如下所示,然后重新启动一下项目。

```java
@RequestMapping(value = "/getUserIdByExtendCommand", method = RequestMethod.GET)
	public ResultMap getUserIdByExtendCommand(@RequestParam Integer id) {
		HelloCommand one = new HelloCommand(restTemplate,id);
		one.execute();
		System.out.println("from cache:   " + one.isResponseFromCache());
	    return new ResultMap(0,"success");
	}
```
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/5.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;这个地方验证了缓存的生命周期只是在这一次请求汇总有效。那么总体上来说Hystrix缓存设置符合一个接口中调用了多次其他接口的时候才会命中。

`# 通过注解的形式

>@CacheResult用来标记缓存结果。@CacheRemove表示清除缓存。


```java
package com.bobo.springcloud.learn.hystrixeurekacache.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.cache.annotation.CacheKey;
import com.netflix.hystrix.contrib.javanica.cache.annotation.CacheRemove;
import com.netflix.hystrix.contrib.javanica.cache.annotation.CacheResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class HelloService implements IHelloService{

	@Autowired
	private RestTemplate restTemplate;

	@Override
	@CacheResult
	@HystrixCommand
	public String hello(Integer id) {
		String json = restTemplate.getForObject("http://HYSTRIX-EUREKA-CACHE-PRODUCER/getUser/{1}", String.class, id);
		System.out.println(json);
		return json;
	}

	@Override
	@CacheResult
	@HystrixCommand(commandKey = "getUser")
	public String getUserToCommandKey(@CacheKey Integer id) {
		String json = restTemplate.getForObject("http://HYSTRIX-EUREKA-CACHE-PRODUCER/getUser/{1}", String.class, id);
		System.out.println(json);
		return json;
	}

	@Override
	@CacheRemove(commandKey="getUser")
	@HystrixCommand
	public String updateUser(@CacheKey Integer id) {
		System.out.println("删除getUser缓存");
		return "update success";
	}
}


    /**
	 * 缓存
	 * @param id
	 * @return
	 */
	@RequestMapping(value = "/getUser/{id}", method = RequestMethod.GET)
	public String getUserId(@PathVariable("id") Integer id) {
		helloService.hello(id);
		helloService.hello(id);
	    return "getUser success";
	}
	
	



```
