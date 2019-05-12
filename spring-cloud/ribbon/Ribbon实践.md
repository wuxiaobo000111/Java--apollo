# Ribbon负载均衡策略

|策略类   |备注  | 描述 |
|---|---|---|
|RandomRule| 随机策略|随机选择server |
|RoundRobinRule| 轮询策略 |按顺序循环选择server|
 |RetryRule |重试策略|在一个配置时间段内当选择server不成功,则一直尝试选择一个可用的server|
|BestAvailableRule |最低并发策略|逐个考察server,如果server断路器打开,则忽略,再选择其中并发连接最低的server|
|AvailabilityFilteringRule |可用过法策略| 过滤掉一直连接失败并被标记为circuit tripped的server,过滤掉那些高并发连接的server (activeconnections超过配置的阈值) |
|ResponseTimeWeightedRule|相应时间加权策略|根据server的响应时间分配权重。响应时间越长,权重越低,被响应时间|选择到的概率就越低;响应时间越短,权重越高,被选择到的概率就越低;相应时间越短,权重越高,被选择到的概率就越高。这个策略很贴切,综合了各种因素,如:网络、磁盘、IO等,这些因素直接影响着响应时间
|ZoneAvoidanceRule |区域权衡策略|综合判断server所在区域的性能和server的可用性轮询选择server,并且判定一个AwsZone的运行性能是否可用,剔除不可用的Zone中的所有server|

# 如何设置负载均衡

## 全局策略设置

```java
package com.bobo.springcloud.learn.ribboneurekademoconsumer.config;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 指定ribbon的负载均衡策略
 */
@Configuration
public class RibbonRuleConfig {

    @Bean
    public IRule iRule () {
        return new RandomRule();
    }
}

```

## 基于注解的策略配置

```java
package com.bobo.springcloud.learn.ribboneurekademoconsumer.config;

public @interface AvoidScan {

}

package com.bobo.springcloud.learn.ribboneurekademoconsumer.config;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 指定ribbon的负载均衡策略
 */
@Configuration
@AvoidScan
public class RibbonRuleConfig {

    @Autowired
    IClientConfig config;

    @Bean
    public IRule ribbonRule(IClientConfig config) {
        return new RandomRule();
    }
}


@RibbonClient(name = "ribbon-eureka-demo-producer", configuration = RibbonRuleConfig.class)
@ComponentScan(excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, value = {AvoidScan.class})})



```

> &nbsp;&nbsp;&nbsp;&nbsp;RibbonClient是对name的服务使用configuration中的配置类。
> @ComponentScan是让Spring不去扫描被@AvoidScan注解标记的配置类。


# Ribbon的超时和重试

```yml
ribbon-eureka-demo-producer:
  ribbon:
    ConnectTimeout: 30000
    ReadTimeout: 30000
    # 对第一次请求的服务的重试次数
    ManAutoRetries: 1
    # 要重试的下一个服务的最大数量。
    MaxAutoRetriesNextServer: 1
    OktoRetryOnAllOpreations: all
```

# Ribbon的饥饿加载

> &nbsp;&nbsp;&nbsp;&nbsp;Ribbon使用的是懒加载。可以通过配置的方式修改为饥饿加载。 

```yml

ribbon:
  eager-load:
    enabled: true
    clients: ribbon-eureka-demo-producer
```
