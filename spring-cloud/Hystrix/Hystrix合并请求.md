# Hystrix合并请求

>&nbsp;&nbsp;&nbsp;&nbsp;当使用请求合并的时候，可以减少并发和请求执行所需要的线程数和网络连接数。

# 使用继承的方式

# 使用注解的方式

## pom依赖

```xml
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

```

## 引入Hystrix上下文

```java
package com.bobo.springcloud.learn.hystrixeurekacollasper.config;

import com.netflix.hystrix.strategy.concurrency.HystrixRequestContext;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HystrixContextInterceptor implements HandlerInterceptor {
	
	private HystrixRequestContext context;
	
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse respone, Object arg2) throws Exception {
		context = HystrixRequestContext.initializeContext();
		return true;
	}

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse respone, Object arg2, ModelAndView arg3)
			throws Exception {
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse respone, Object arg2, Exception arg3)
			throws Exception {
		context.shutdown();
	}
	
}


package com.bobo.springcloud.learn.hystrixeurekacollasper.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.mvc.Controller;

@Configuration
public class CollapsingConfiguration {
	
    @Bean
    @ConditionalOnClass(Controller.class)
    public HystrixContextInterceptor userContextInterceptor() {
        return new HystrixContextInterceptor();
    }

    @Configuration
    @ConditionalOnClass(Controller.class)
    public class WebMvcConfig extends WebMvcConfigurerAdapter {
        @Autowired
        HystrixContextInterceptor userContextInterceptor;
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(userContextInterceptor);
        }
    }
}

```

## 接口信息

```java
package com.bobo.springcloud.learn.hystrixeurekacollasper.service;

public class Animal {
	private String name;
	private String sex;
	private int age;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getSex() {
		return sex;
	}
	public void setSex(String sex) {
		this.sex = sex;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public Animal() {}
	public Animal(String name, String sex, int age) {
		this.name = name;
		this.sex = sex;
		this.age = age;
	}
	
}

package com.bobo.springcloud.learn.hystrixeurekacollasper.service;


import java.util.concurrent.Future;

public interface ICollapsingService {
	public Future<Animal> collapsing(Integer id);
	public Animal collapsingSyn(Integer id);
	public Future<Animal> collapsingGlobal(Integer id);
	
}


package com.bobo.springcloud.learn.hystrixeurekacollasper.service;

import com.netflix.hystrix.HystrixCollapser.Scope;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCollapser;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Future;

@Component
public class CollapsingService implements ICollapsingService {

    @Override
    @HystrixCollapser(batchMethod = "collapsingList", collapserProperties = {
            @HystrixProperty(name = "timerDelayInMilliseconds", value = "1000")
    })
    public Future<Animal> collapsing(Integer id) {
        return null;
    }

    @Override
    @HystrixCollapser(batchMethod = "collapsingListGlobal", scope = Scope.GLOBAL, collapserProperties = {
            @HystrixProperty(name = "timerDelayInMilliseconds", value = "10000")
    })
    public Future<Animal> collapsingGlobal(Integer id) {
        return null;
    }


    @Override
    @HystrixCollapser(batchMethod = "collapsingList", collapserProperties = {
            @HystrixProperty(name = "timerDelayInMilliseconds", value = "1000")
    })
    public Animal collapsingSyn(Integer id) {
        // TODO Auto-generated method stub
        return null;
    }

    @HystrixCommand
    public List<Animal> collapsingList(List<Integer> animalParam) {
        System.out.println("collapsingList当前线程" + Thread.currentThread().getName());
        System.out.println("当前请求参数个数:" + animalParam.size());
        List<Animal> animalList = new ArrayList<Animal>();
        for (Integer animalNumber : animalParam) {
            Animal animal = new Animal();
            animal.setName("Cat - " + animalNumber);
            animal.setSex("male");
            animal.setAge(animalNumber);
            animalList.add(animal);
        }
        return animalList;
    }


    @HystrixCommand
    public List<Animal> collapsingListGlobal(List<Integer> animalParam) {
        System.out.println("collapsingListGlobal当前线程" + Thread.currentThread().getName());
        System.out.println("当前请求参数个数:" + animalParam.size());
        List<Animal> animalList = new ArrayList<Animal>();
        for (Integer animalNumber : animalParam) {
            Animal animal = new Animal();
            animal.setName("Dog - " + animalNumber);
            animal.setSex("male");
            animal.setAge(animalNumber);
            animalList.add(animal);
        }
        return animalList;
    }

}



```

>&nbsp;&nbsp;&nbsp;&nbsp; @HystrixCollapser表示是请求合并。batchMethod属性是指定
要执行的方法。timerDelayInMilliseconds表示合并多少ms之内的情感求。默认是10ms。需要注意的是
Feign不支持请求合并。

>&nbsp;&nbsp;&nbsp;&nbsp;如果请求两次接口是再不同线程中运行的。那么需要指定 scope = Scope.GLOBAL。
默认情况下Request。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/6.jpg?raw=true)