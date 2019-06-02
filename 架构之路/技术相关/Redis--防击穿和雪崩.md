>来自高可用架构「ArchNotes」微信公众号
>参考博客https://www.jianshu.com/p/81699cb41523

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/56.jpg?raw=true)


# 缓存击穿预防以及优化

>&nbsp;&nbsp;&nbsp;&nbsp;缓存穿透是指查询一个根本不存在的数据，缓存层和存储层都不会命中，但是出于容错的考虑，如果从存储层查不到数据则不写入缓存层，如下图所示整个过程分为如下 3 步：

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/57.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;缓存穿透问题可能会使后端存储负载加大，由于很多后端存储不具备高并发性，甚至可能造成后端存储宕掉。通常可以在程序中分别统计总调用数、缓存层命中数、存储层命中数，如果发现大量存储层空命中，可能就是出现了缓存穿透问题。
&nbsp;&nbsp;&nbsp;&nbsp;造成缓存穿透的基本有两个。第一，业务自身代码或者数据出现问题，第二，一些恶意攻击、爬虫等造成大量空命中，下面我们来看一下如何解决缓存穿透问题。

## 解决方式

### 缓存空对象

>&nbsp;&nbsp;&nbsp;&nbsp;如下图所示，当第 2 步存储层不命中后，仍然将空对象保留到缓存层中，之后再访问这个数据将会从缓存中获取，保护了后端数据源。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/58.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;缓存空对象会有两个问题：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一，空值做了缓存，意味着缓存层中存了更多的键，需要更多的内存空间 ( 如果是攻击，问题更严重 )，比较有效的方法是针对这类数据设置一个较短的过期时间，让其自动剔除。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第二，缓存层和存储层的数据会有一段时间窗口的不一致，可能会对业务有一定影响。例如过期时间设置为 5 分钟，如果此时存储层添加了这个数据，那此段时间就会出现缓存层和存储层数据的不一致，此时可以利用消息系统或者其他方式清除掉缓存层中的空对象。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/59.jpg?raw=true)

### 使用布隆过滤器

>&nbsp;Bloom Filter（BF）是一种空间效率很高的随机数据结构，它利用位数组很简洁地表示一个集合，并能判断一个元素是否属于这个集合。它是一个判断元素是否存在集合的快速的概率算法。Bloom Filter有可能会出现错误判断，但不会漏掉判断。也就是Bloom Filter判断元素不再集合，那肯定不在。如果判断元素存在集合中，有一定的概率判断错误。因此，Bloom Filter不适合那些“零错误”的应用场合。而在能容忍低错误率的应用场合下，Bloom Filter比其他常见的算法（如hash，折半查找）极大节省了空间。

### 示例代码

```xml
<dependencies>
    <dependency>
        <groupId>com.google.guava</groupId>     
        <artifactId>guava</artifactId>      
        <version>22.0</version>
    </dependency>
</dependencies>
```

```java
import java.util.ArrayList;import java.util.List; 
import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels; 

public class Test {
    
    private static int size = 1000000;
    
        private static BloomFilter<Integer> bloomFilter =BloomFilter.create(Funnels.integerFunnel(), size);
            public static void main(String[] args) {        
            for (int i = 0; i < size; i++) {            
            bloomFilter.put(i);        
        }        
        
        List<Integer> list = new ArrayList<Integer>(1000);
        //故意取10000个不在过滤器里的值，看看有多少个会被认为在过滤器里
        for (int i = size + 10000; i < size + 20000; i++) {
            if (bloomFilter.mightContain(i)) {
                list.add(i);
            }
        }
        System.out.println("误判的数量：" + list.size());
    }
}

```


>&nbsp;&nbsp;&nbsp;&nbsp;redis伪代码

```java
String get(String key) {
    String value = redis.get(key);     
    if (value  == null) {
        if(!bloomfilter.mightContain(key)){
            return null; 
        }else{
            value = db.get(key); 
            redis.set(key, value); 
        }    
    }
    return value；
}
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/60.jpg?raw=true)

# 缓存雪崩优化

>&nbsp;&nbsp;&nbsp;&nbsp;由于缓存层承载着大量请求，有效的保护了存储层，但是如果缓存层由于某些原因整体不能提供服务，于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。 缓存雪崩的英文原意是 stampeding herd（奔逃的野牛），指的是缓存层宕掉后，流量会像奔逃的野牛一样，打向后端存储。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/61.jpg?raw=true)

## 解决方案



### 保证缓存的高可用

>&nbsp;&nbsp;&nbsp;&nbsp;和飞机都有多个引擎一样，如果缓存层设计成高可用的，即使个别节点、个别机器、甚至是机房宕掉，依然可以提供服务，例如前面介绍过的 Redis Sentinel 和 Redis Cluster 都实现了高可用。


### 使用隔离组件对后端服务进行限流

>&nbsp;&nbsp;&nbsp;&nbsp;无论是缓存层还是存储层都会有出错的概率，可以将它们视同为资源。作为并发量较大的系统，假如有一个资源不可用，可能会造成线程全部 hang 在这个资源上，造成整个系统不可用。降级在高并发系统中是非常正常的：比如推荐服务中，如果个性化推荐服务不可用，可以降级补充热点数据，不至于造成前端页面是开天窗。


### 提前演练。

>&nbsp;&nbsp;&nbsp;&nbsp;在项目上线前，演练缓存层宕掉后，应用以及后端的负载情况以及可能出现的问题，在此基础上做一些预案设定。


# 缓存热点 key 重建优化

>&nbsp;&nbsp;&nbsp;&nbsp;开发人员使用缓存 + 过期时间的策略既可以加速数据读写，又保证数据的定期更新，这种模式基本能够满足绝大部分需求。但是有两个问题如果同时出现，可能就会对应用造成致命的危害：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当前 key 是一个热点 key( 例如一个热门的娱乐新闻），并发量非常大。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;重建缓存不能在短时间完成，可能是一个复杂计算，例如复杂的 SQL、多次 IO、多个依赖等。
&nbsp;&nbsp;&nbsp;&nbsp;在缓存失效的瞬间，有大量线程来重建缓存 ( 如下图)，造成后端负载加大，甚至可能会让应用崩溃。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/62.jpg?raw=true)

## 解决方案

### 使用互斥锁

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/63.jpg?raw=true)


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/64.jpg?raw=true)


### 永远不过期

>&nbsp;&nbsp;&nbsp;&nbsp;“永远不过期”包含两层意思：

```text
从缓存层面来看，确实没有设置过期时间，所以不会出现热点 key 过期后产生的问题，也就是“物理”不过期。


从功能层面来看，为每个 value 设置一个逻辑过期时间，当发现超过逻辑过期时间后，会使用单独的线程去构建缓存。
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/65.jpg?raw=true)


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/66.jpg?raw=true)