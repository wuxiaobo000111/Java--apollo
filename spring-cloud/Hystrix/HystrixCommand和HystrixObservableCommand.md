# HystrixCommand

> &nbsp;&nbsp;&nbsp;&nbsp;封装执行的代码,然后具有故障延迟容错、断路器和统计等功能。但是这个是个阻塞命令。

## 如何使用

```java
package com.bobo.springcloud.learn.hystrixeurekaexception.service;

import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.exception.HystrixBadRequestException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class FallbackBadRequestExpcetionService extends HystrixCommand<String>{


	private static Logger log = LoggerFactory.getLogger(FallbackBadRequestExpcetionService.class);

    public FallbackBadRequestExpcetionService() {
        super(HystrixCommandGroupKey.Factory.asKey("GroupBRE"));
    }


	@Override
	protected String run() {
		 throw new HystrixBadRequestException("HystrixBadRequestException error");
	}
	
    @Override
    protected String getFallback() {
    	System.out.println(getFailedExecutionException().getMessage());
        return "invoke HystrixBadRequestException fallback method:  ";
    }
	
	
}

package com.bobo.springcloud.learn.hystrixeurekaexception.controller;

import com.bobo.springcloud.learn.hystrixeurekaexception.service.FallbackBadRequestExpcetionService;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ExceptionController {

    private Logger logger = LoggerFactory.getLogger(ExceptionController.class);


    @GetMapping("/getPSFallbackBadRequestExpcetion")
    public String providerServiceFallback(){
        String result = new FallbackBadRequestExpcetionService().execute();
        return result;
    }


    @GetMapping("/getFallbackMethodTest")
    @HystrixCommand
    public String getFallbackMethodTest(String id){
        throw new RuntimeException("getFallbackMethodTest failed");
    }

    public String fallback(String id, Throwable throwable) {
        logger.error(throwable.getMessage());
        return "this is fallback message";
    }

}
```

# HystrixObservableCommand
## 如何使用

```java
package com.bobo.springcloud.learn.hystrixeurekathread.service;

import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixObservableCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;
import rx.Observable;
import rx.Subscriber;
import rx.schedulers.Schedulers;

public class ThreadContextSyncService extends HystrixObservableCommand<String> {

    @Autowired
    private RestTemplate restTemplate;

    private String name;

    public ThreadContextSyncService(String name) {
        // 调用父类构造方法
        super(HystrixCommandGroupKey.Factory.asKey("usercommand"));
        this.name = name;
    }

    @Override
    protected Observable<String> construct() {
       return Observable.create(new Observable.OnSubscribe<String>() {
           @Override
           public void call(Subscriber<? super String> subscriber) {
                try {
                    //伪造超时事件
                    Thread.sleep(10000);
                    String template = restTemplate.getForObject("http://FEIGN-EUREKA-HYSTRIX-PRODUCER/hello?name={1}",
                            String.class, name);
                    subscriber.onNext(template);
                    subscriber.onCompleted();
                }catch (Exception e) {
                    subscriber.onError(e);
                }
           }
       }).subscribeOn(Schedulers.io());
    }

    @Override
    protected Observable<String> resumeWithFallback() {
        return Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                try {
                    subscriber.onNext("error");
                    subscriber.onCompleted();
                }catch (Exception e) {
                    subscriber.onError(e);
                }
            }
        }).subscribeOn(Schedulers.io());
    }
}


package com.bobo.springcloud.learn.hystrixeurekathread.controller;

import com.bobo.springcloud.learn.hystrixeurekathread.service.ThreadContextSyncService;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import rx.Observable;
import rx.Observer;

/**
 * @author wuxiaobo
 */

@RestController
public class HelloController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String hello(String name) {
        final String[] result = {""};
        ThreadContextSyncService threadContextSyncService = new ThreadContextSyncService(name);
        Observable<String> observe = threadContextSyncService.observe();
        observe.subscribe(new Observer<String>() {
            @Override
            public void onCompleted() {
                System.out.println("聚合完了所有的查询请求!");
            }

            @Override
            public void onError(Throwable t) {
                t.printStackTrace();
            }

            @Override
            public void onNext(String s) {
                System.out.println("hello result =" +s);
            }
        });
        return "success";
    }
}


```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/8.jpg?raw=true)


# 共同点和不同点

```text
Hystrix在使用过程中除了HystrixCommand,还有HystrixObservableCommand,这两个命令有很多共同点,如都支持故障和延迟容错、断路器、指标统计。
但两者也有很多区别:
	1) HystrixCommand默认是阻塞式的,可以提供同步和异步两种方式,但HystrixObserva-leCommand是非阻塞的,默认只能是异步的。
    2) HystrixCommand的方法是run, HystrixObservableCommand执行的是construct.
	3) HystrixCommand一个实例一次只能发一条数据出去, HystrixObservableCommand可以发送多条数据。
```

# 属性

```text
    commandkey:全局唯一的标识符,如果不配则默认是方法名
	defaultFallback :默认的fallback方法,该函数不能有入参,返回值和方法保持一致,但fallbackMethod优先级更高。
	fallbackMethod :指定的处理回退逻辑的方法,必须和HystrixCommand在同一个类里,方法的参数要保持一致。
	ignoreExceptions: HystrixBadRequestException不会触发fallback,这里定义的就是你不希望哪些异常被fallback而是直接抛出。
    commandProperties:配置一些命名的属性,如执行的隔离策略等
    threadPoolProperties:用来配置线程池相关的属性。
	groupKey:全局唯一标识服务分组的名称,内部会根据这个兼职来展示统计数、仪表盘等信息,默认的线程划分是根据这命令组的名称来进行的,一般会在创建
	    HystrixCommond时指定命令组来实现默认的线程池划分。
	threadPoolKey:对服务的线程池信息进行设置,用于HystrixThreadPool监控、metrics、缓存等用途。

```