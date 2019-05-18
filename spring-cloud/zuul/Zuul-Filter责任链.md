# Zuul Filter链的工作原理

## Filter主要特性


```text
    1.Filter的类型: Filter的类型决定了此Filter在Filter链中的执行顺序。可能是路由动作发生前,可能是路由动作发生时,
    可能是路由动作发生后,也可能是路由过程发生异常时。
    2.Filter的执行顺序:同一种类型的Filter可以通过fiterOrder()方法来设定执行顺序。一般会根据业务的执行顺序需求,
    来设定自定义Filter的执行顺序。
    3.Filter的执行条件: Filter运行所需要的标准或条件。
    4.Filter的执行效果:符合某个Filter执行条件,产生的执行效果。

    Zuul内部提供了一个动态读取、编译和运行这些Filter的机制。Filter之间不直接通信,在请求线程中会通过
    RequestContext来共享状态,它的内部是用ThreadLocal实现的,当然你也可以在Filter之间使用ThreadLocal
    来收集自己需要的状态或数据。

```

>&nbsp;在ZuulServlet类中定义了Filter的执行逻辑.

```java
        @Override
        public void service(javax.servlet.ServletRequest servletRequest, javax.servlet.ServletResponse servletResponse) throws ServletException, IOException {
        try {
            init((HttpServletRequest) servletRequest, (HttpServletResponse) servletResponse);
            RequestContext context = RequestContext.getCurrentContext();
            context.setZuulEngineRan();

            try {
                preRoute();
            } catch (ZuulException e) {
                error(e);
                postRoute();
                return;
            }
            try {
                route();
            } catch (ZuulException e) {
                error(e);
                postRoute();
                return;
            }
            try {
                postRoute();
            } catch (ZuulException e) {
                error(e);
                return;
            }

        } catch (Throwable e) {
            error(new ZuulException(e, 500, "UNHANDLED_EXCEPTION_" + e.getClass().getName()));
        } finally {
            RequestContext.getCurrentContext().unset();
        }
    }
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/22.png?raw=true)


>&nbsp;Zuul一共有四种不同声明周期的Filter,如下所示:

```text
    1.pre:在Zuul按照规则路由到下级服务之前执行。如果需要对请求进行预处理,比如鉴权、限流等,都应考虑在此类Filter实现。
    2.route:这类Filter是Zuul路由动作的执行者,是Apache HttpClient或Netlix Ribbon构建和发送原始HTTP请求的地方
    ,目前已支持OkHttpo
    3.post:这类Filter是在源服务返回结果或者异常信息发生后执行的,如果需要对返回信息做一些处理,则在此类Filter进行处理
    4.error:在整个生命周期内如果发生异常,则会进入error Filter,可做全局异常处理
```


>&nbsp;&nbsp;&nbsp;&nbsp;在实际项目中,往往需要自实现以上类型的Filter来对请求链路进行处理,根据业务的需求,选取相应生命周期的Filter来达成目的。在Filter之间,通过com.netflix.zuul.context.RequestContext类来进行通信,内部采用ThreadLocal保存每个请求的一些信息,包括请求路由、错误信息、HttpServletRequest,HttpServletResponse,这使得一些操作是十分可靠的,它还扩展了ConcurrentHashMap, 目的是为了在处理过程中保存任何形式的信息。

## 原生Filter

### 原生Filter中使用actuator

#### 引入依赖

```xml

  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

#### 暴露端点

```yml

management:
  endpoints:
    web:
      exposure:
        include: "*"

```

>&nbsp;使用http://127.0.0.1:8102/actuator/routes/details可以返回当前Zuul Server中所有已经生成的规则。使用http://127.0.0.1:8102/actuator/filters可以查询所有已经注册的Filter。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/23.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/24.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;通过上图我们可以得到Zuul内置的Filter的类型。

#### pre Filter

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/25.jpg?raw=true)

#### route Filter

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/26.jpg?raw=true)


#### post Filter

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/27.jpg?raw=true)

#### error Filter 

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/28.jpg?raw=true)

>如股票要禁用某一个filter。则可以使用zuul.<ClassName>.<filterType>.disable=false.


## 如何自定义个最简单的Zuul Filter

>&nbsp;&nbsp;&nbsp;&nbsp;这里的实现其实是非常简单继承ZuulFilter这个抽象类。代码如下所示


```java
package com.bobo.springcloud.learn.zuuleurekaserverdemo.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;

public class FirstPreFilter extends ZuulFilter {

    /**
     * Filter的类型
     * @return
     */
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    /**
     * Filter的执行次序
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 使用返回值来设置该Filter是否执行。可以作为开关来使用
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 业务逻辑
     * @return
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {
        System.out.println("hello, I am First Zuul Filter");
        return null;
    }
}


package com.bobo.springcloud.learn.zuuleurekaserverdemo.config;

import com.bobo.springcloud.learn.zuuleurekaserverdemo.filter.FirstPreFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ZuulFilterConfig {

    @Bean
    public FirstPreFilter firstPreFilter () {
        return new FirstPreFilter();
    }
}

```

>&nbsp;&nbsp;&nbsp;&nbsp;重启项目,然后访问接口。会在控制台输出。
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/29.jpg?raw=true)、

### 如何使用上下文

```java

package com.bobo.springcloud.learn.zuuleurekaserverdemo.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;

import javax.servlet.http.HttpServletRequest;

import static org.springframework.cloud.netflix.zuul.filters.support.FilterConstants.PRE_TYPE;

public class SecondPreFilter extends ZuulFilter {

	@Override
	public String filterType() {
		return PRE_TYPE;
	}

	@Override
	public int filterOrder() {
		return 2;
	}

	@Override
	public boolean shouldFilter() {
		return true;
	}

	@Override
	public Object run() throws ZuulException {
		System.out.println("这是SecondPreFilter！");
		//从RequestContext获取上下文
		RequestContext ctx = RequestContext.getCurrentContext();
		//从上下文获取HttpServletRequest
		HttpServletRequest request = ctx.getRequest();
		//从request尝试获取a参数值
		String a = request.getParameter("a");
		//如果a参数值为空则进入此逻辑
		if (null == a) {
			//对该请求禁止路由，也就是禁止访问下游服务
			ctx.setSendZuulResponse(false);
			//设定responseBody供PostFilter使用
			ctx.setResponseBody("{\"status\":500,\"message\":\"a参数为空！\"}");
			//logic-is-success保存于上下文，作为同类型下游Filter的执行开关
			ctx.set("logic-is-success", false);
			//到这里此Filter逻辑结束
			return null;
		}
		//设置避免报空
		ctx.set("logic-is-success", true);
		return null;
	}
}


package com.bobo.springcloud.learn.zuuleurekaserverdemo.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;

import javax.servlet.http.HttpServletRequest;

import static org.springframework.cloud.netflix.zuul.filters.support.FilterConstants.PRE_TYPE;

public class ThirdPreFilter extends ZuulFilter {
	
	@Override
	public String filterType() {
		return PRE_TYPE;
	}
	
	@Override
	public int filterOrder() {
		return 3;
	}

	@Override
	public boolean shouldFilter() {
		RequestContext ctx = RequestContext.getCurrentContext();
		//从上下文获取logic-is-success值，用于判断此Filter是否执行
		return (boolean)ctx.get("logic-is-success");
	}

	@Override
	public Object run() throws ZuulException {
		System.out.println("这是ThirdPreFilter！");
		//从RequestContext获取上下文
		RequestContext ctx = RequestContext.getCurrentContext();
		//从上下文获取HttpServletRequest
		HttpServletRequest request = ctx.getRequest();
		//从request尝试获取b参数值
		String b = request.getParameter("b");
		//如果b参数值为空则进入此逻辑
		if (null == b) {
			//对该请求禁止路由，也就是禁止访问下游服务
			ctx.setSendZuulResponse(false);
			//设定responseBody供PostFilter使用
			ctx.setResponseBody("{\"status\":500,\"message\":\"b参数为空！\"}");
			//logic-is-success保存于上下文，作为同类型下游Filter的执行开关，假定后续还有自定义Filter当设置此值
			ctx.set("logic-is-success", false);
			//到这里此Filter逻辑结束
			return null;
		}
		return null;
	}
}


  @Bean
    public SecondPreFilter secondPreFilter () {
        return new SecondPreFilter();
    }

    @Bean
    public ThirdPreFilter thirdPreFilter () {
        return new ThirdPreFilter();
    }

    上述代码来自于《重新定义Spring Cloud实战》一书。
```

