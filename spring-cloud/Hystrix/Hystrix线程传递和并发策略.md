# Hystrix线程传递和并发策略

>&nbsp;&nbsp;&nbsp;&nbsp;Hystrix提供了两种隔离模式和进行请求的操作。一种是信号量,一种是线程隔离。
如果使用信号量,则Hystrix在请求的时候会获取一个信号量。如果成功拿到,则请求继续执行。请求在一个线程汇总执行
完毕。如果是线程隔离,Hystrix会把请求放在线程池中执行。导致线程一的上下文数据在
线程池二中拿不到。首先让我们看一下错误的代码写法。


## 错误写法

### pom依赖

```java
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
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

## service层

```java
package com.bobo.springcloud.learn.hystrixeurekathread.service;

public interface IThreadContextService {
	public String getUser(Integer id);
	
}

package com.bobo.springcloud.learn.hystrixeurekathread.service;

import com.bobo.springcloud.learn.hystrixeurekathread.Hystrix.HystrixThreadLocal;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;

@Component

public class ThreadContextService implements IThreadContextService {

    private static final Logger log = LoggerFactory.getLogger(ThreadContextService.class);

    @Override
    @HystrixCommand
    public String getUser(Integer id) {
        log.info("ThreadContextService, Current thread : " + Thread.currentThread().getId());

        log.info("ThreadContextService, ThreadContext object : " + HystrixThreadLocal.threadLocal.get());
        log.info("ThreadContextService, RequestContextHolder : " + RequestContextHolder.currentRequestAttributes().getAttribute("userId", RequestAttributes.SCOPE_REQUEST).toString());
        return "Success";
    }


}


```

### 接口

```java
package com.bobo.springcloud.learn.hystrixeurekathread.controller;

import com.bobo.springcloud.learn.hystrixeurekathread.Hystrix.HystrixThreadLocal;
import com.bobo.springcloud.learn.hystrixeurekathread.service.IThreadContextService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;

@RestController
public class ThreadContextController {
    private static final Logger log = LoggerFactory.getLogger(ThreadContextController.class);
	   
	@Autowired
	private IThreadContextService threadContextService;
	
	@RequestMapping(value = "/getUser/{id}", method = RequestMethod.GET)
    public String getUser(@PathVariable("id") Integer id) {
		//第一种测试，放入上下文对象
		HystrixThreadLocal.threadLocal.set("userId : "+ id);
		//第二种测试，利用RequestContextHolder放入对象测试
		RequestContextHolder.currentRequestAttributes().setAttribute("userId", "userId : "+ id, RequestAttributes.SCOPE_REQUEST);
		log.info("ThreadContextController, Current thread: " + Thread.currentThread().getId());
        log.info("ThreadContextController, Thread local: " + HystrixThreadLocal.threadLocal.get());
        log.info("ThreadContextController, RequestContextHolder: " + RequestContextHolder.currentRequestAttributes().getAttribute("userId", RequestAttributes.SCOPE_REQUEST));
        //调用
        String user = threadContextService.getUser(id);
        return user;
    }
}
```

### 启动类和配置文件

```java
package com.bobo.springcloud.learn.hystrixeurekathread;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@EnableDiscoveryClient
@EnableHystrix
@EnableHystrixDashboard
@SpringBootApplication
public class HystrixEurekaThreadApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixEurekaThreadApplication.class, args);
    }

}

```

```yml
server:
  port: 8099
eureka:
  client:
    serviceUrl:
      defaultZone: http://192.168.88.128:8761/eureka/,http://192.168.88.128:8760/eureka/
management:
  endpoints:
    web:
      exposure:
        include: "*"
spring:
  application:
    name: hystrix-eureka-thread



```

> 启动项目访问结果。
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/7.jpg?raw=true)


> &nbsp;&nbsp;&nbsp;&nbsp;首先我们在Controller代码中在RequestContextHolder放入了一个对象。然后在service层的方式是交给了
> Hystrix去管理的,在取数据的时候报错。

> &nbsp;&nbsp;&nbsp;&nbsp;要解决此类问题有两种方式。

```text
    1.通过修改Hystrix的隔离策略。在Hystrix中默认使用的线程池隔离。可以修改为信号量来管理Hystrix。
    2.通过使用HystrixConcurrencyStrategy。
   
```


# HystrixConcurrencyStrategy

```java

package com.bobo.springcloud.learn.hystrixeurekathread.Hystrix;

import com.bobo.springcloud.learn.hystrixeurekathread.Hystrix.HystrixThreadLocal;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;

import java.util.concurrent.Callable;

public class HystrixThreadCallable<S> implements Callable<S>{
	 private final RequestAttributes requestAttributes;  
	 private final Callable<S> delegate;
	 private String params;
	  
     public HystrixThreadCallable(Callable<S> callable, RequestAttributes requestAttributes,String params) {  
         this.delegate = callable; 
         this.requestAttributes = requestAttributes;  
         this.params = params;
     }

     @Override  
     public S call() throws Exception {  
         try {
             RequestContextHolder.setRequestAttributes(requestAttributes);
             HystrixThreadLocal.threadLocal.set(params);
             return delegate.call();  
         } finally {
             RequestContextHolder.resetRequestAttributes();
             HystrixThreadLocal.threadLocal.remove();
         }  
     }  
       
}

package com.bobo.springcloud.learn.hystrixeurekathread.Hystrix;

import com.netflix.hystrix.HystrixThreadPoolKey;
import com.netflix.hystrix.HystrixThreadPoolProperties;
import com.netflix.hystrix.strategy.HystrixPlugins;
import com.netflix.hystrix.strategy.concurrency.HystrixConcurrencyStrategy;
import com.netflix.hystrix.strategy.concurrency.HystrixRequestVariable;
import com.netflix.hystrix.strategy.concurrency.HystrixRequestVariableLifecycle;
import com.netflix.hystrix.strategy.eventnotifier.HystrixEventNotifier;
import com.netflix.hystrix.strategy.executionhook.HystrixCommandExecutionHook;
import com.netflix.hystrix.strategy.metrics.HystrixMetricsPublisher;
import com.netflix.hystrix.strategy.properties.HystrixPropertiesStrategy;
import com.netflix.hystrix.strategy.properties.HystrixProperty;
import org.springframework.web.context.request.RequestContextHolder;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.Callable;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class SpringCloudHystrixConcurrencyStrategy extends HystrixConcurrencyStrategy {
    private HystrixConcurrencyStrategy delegateHystrixConcurrencyStrategy;
    
    @Override
    public <T> Callable<T> wrapCallable(Callable<T> callable) {
        return new HystrixThreadCallable<>(callable, RequestContextHolder.getRequestAttributes(),HystrixThreadLocal.threadLocal.get());
    }
    
    public SpringCloudHystrixConcurrencyStrategy() {
    	init();
    }
    
    private void init() {
   	 try {
            this.delegateHystrixConcurrencyStrategy = HystrixPlugins.getInstance().getConcurrencyStrategy();
            if (this.delegateHystrixConcurrencyStrategy instanceof SpringCloudHystrixConcurrencyStrategy) {
                return;
            }

            HystrixCommandExecutionHook commandExecutionHook = HystrixPlugins.getInstance().getCommandExecutionHook();
            HystrixEventNotifier eventNotifier = HystrixPlugins.getInstance().getEventNotifier();
            HystrixMetricsPublisher metricsPublisher = HystrixPlugins.getInstance().getMetricsPublisher();
            HystrixPropertiesStrategy propertiesStrategy = HystrixPlugins.getInstance().getPropertiesStrategy();

            HystrixPlugins.reset();
            HystrixPlugins.getInstance().registerConcurrencyStrategy(this);
            HystrixPlugins.getInstance().registerCommandExecutionHook(commandExecutionHook);
            HystrixPlugins.getInstance().registerEventNotifier(eventNotifier);
            HystrixPlugins.getInstance().registerMetricsPublisher(metricsPublisher);
            HystrixPlugins.getInstance().registerPropertiesStrategy(propertiesStrategy);
        }
        catch (Exception e) {
            throw e;
        }
   }
    
    
    @Override
	public ThreadPoolExecutor getThreadPool(HystrixThreadPoolKey threadPoolKey,
			HystrixProperty<Integer> corePoolSize,
			HystrixProperty<Integer> maximumPoolSize,
			HystrixProperty<Integer> keepAliveTime, TimeUnit unit,
			BlockingQueue<Runnable> workQueue) {
		return this.delegateHystrixConcurrencyStrategy.getThreadPool(threadPoolKey, corePoolSize, maximumPoolSize,
				keepAliveTime, unit, workQueue);
	}

	@Override
	public ThreadPoolExecutor getThreadPool(HystrixThreadPoolKey threadPoolKey,
			HystrixThreadPoolProperties threadPoolProperties) {
		return this.delegateHystrixConcurrencyStrategy.getThreadPool(threadPoolKey, threadPoolProperties);
	}

	@Override
	public BlockingQueue<Runnable> getBlockingQueue(int maxQueueSize) {
		return this.delegateHystrixConcurrencyStrategy.getBlockingQueue(maxQueueSize);
	}

	@Override
	public <T> HystrixRequestVariable<T> getRequestVariable(
			HystrixRequestVariableLifecycle<T> rv) {
		return this.delegateHystrixConcurrencyStrategy.getRequestVariable(rv);
	}

}


package com.bobo.springcloud.learn.hystrixeurekathread.Hystrix;

import com.bobo.springcloud.learn.hystrixeurekathread.Hystrix.HystrixThreadLocal;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;

import java.util.concurrent.Callable;

public class HystrixThreadCallable<S> implements Callable<S>{
	 private final RequestAttributes requestAttributes;  
	 private final Callable<S> delegate;
	 private String params;
	  
     public HystrixThreadCallable(Callable<S> callable, RequestAttributes requestAttributes,String params) {  
         this.delegate = callable; 
         this.requestAttributes = requestAttributes;  
         this.params = params;
     }

     @Override  
     public S call() throws Exception {  
         try {
             RequestContextHolder.setRequestAttributes(requestAttributes);
             HystrixThreadLocal.threadLocal.set(params);
             return delegate.call();  
         } finally {
             RequestContextHolder.resetRequestAttributes();
             HystrixThreadLocal.threadLocal.remove();
         }  
     }  
       
}




```

> &nbsp;&nbsp;&nbsp;&nbsp;具体上面代码是神马意思，我也不是很请求,知道这是种解决方案就行。


