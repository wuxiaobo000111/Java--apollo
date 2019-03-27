
# 查看所有的键

    keys *

# 键总数
    dbsize

# 检查键是否存在
    exists key


# 删除键

del key1 key2 key3 [key......]

# 单个键的管理

##  键过期

expire key seconds 

>&nbsp;&nbsp;&nbsp;&nbsp;ttl会返回键的剩余过期时间

```text
大于等于0的整数:键剩余的过期时间
-1: 没有设置过期时间
-2: 键不存在
```

##  键的数据结构类型

type key

## 键重命名

```redis
rename key newKey

注意:如果newkey之前已经存在,那么这个newkey原来的值也会被覆盖


redis提供了renamenx命令,确保只有newkey不存在的时候才会被覆盖。例子如下所示:

重命名键期间会执行del命令删除旧的键,如果键对应的值比较大,会存在阻塞redis的可能性。

```
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/3.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/4.jpg?raw=true)

## 键过期

>&nbsp;&nbsp;&nbsp;&nbsp;在redis中,除去使用expire,ttl命令之外,redis还提供了expireat,pexpire,pexpireat,pttl,persist等命令。


```redis
expire key seconds:键在seconds秒之后过期。

exprieat key timestamp: 键在秒级时间戳timestamp后过期
```
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/5.jpg?raw=true)


```redis
ttl:查询键的剩余过期时间,精确到秒级。
pttl: 查询键的剩余过期时间,精确在毫秒级别。

两者都会有返回值:
    大于等于0的整数:键剩余的过期时间。
    -1:键没有设置过期时间
    -2:键不存在
```
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/6.jpg?raw=true)


```redis
redis还提供了毫秒级的过期方案:
    pexpire key milliseconds : 键在milliseconds毫秒之后过期。
    pexpireat key milliseconds-timestamp: 键在毫秒级时间戳之后才过期


 无论是哪种过期行为,redis的最内部都是使用的是pexpireat。   
```

>&nbsp;&nbsp;&nbsp;&nbsp;redis设置过期命令需要注意一下几点

```redis
1.如果expire key的键不存在,那么返回值就是0.
2.如果过期时间为负值,键则会立即删除,就好像使用了del命令一样。
3.persist命令可以将键的过期时间清除。

4.对于字符串类型,执行set命令会去掉过期时间
5.redis不支持二级数据结构内部元素的过期时间,比如不能对列表类型的一个元素做过期时间的设置。
6.setex命令作为set+expire的组合,不仅仅是原子操作,而且减少了一次网络通过的时间。或者是直接在
set的时候就设置过期时间。
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/8.jpg?raw=true)


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/9.jpg?raw=true)


##  迁移键

>&nbsp;&nbsp;&nbsp;&nbsp;redis在发展过程中提供了move,dump+restore,migrate三种迁移方式


### move

>&nbsp;&nbsp;&nbsp;&nbsp;在redis内部进行迁移,比如说从第一个库迁移到了第二个库。
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/10.jpg?raw=true)


### dump+restore

```redis

    dump key 
    restore key ttl value
```

>&nbsp;&nbsp;&nbsp;&nbsp;这个命令实现了不同的redis实例之间进行数据迁移的功能。整个功能会被分成为两个部分
```text
1.在源redis上,dump命令会将键值序列化。采用的格式是RDB格式。
2.在目标redis上,restore命令将上面序列化的值进行复原。
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/11.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;需要注意两个地方:

```text
1.这个操作并非是原子性的,而是通过客户端分步完成的。
2.迁移过程启动了两个客户端连接,所以dump不是在源redis和目标redis中进行传输的。
```


### migrate

``` redis
MIGRATE host port key|"" destination-db timeout [COPY] [REPLACE] [KEYS key [key ...]] 


host： 目标redis的IP地址
port:  目标redis的端口
key|"" 如果要迁移一个键,那么这个地方就是key.如果迁移多个键,
则这个地方就是""。需要注意的是这里和redis的版本有关。迁移多个键需要在redis3.0.6之后才支持。
destination-db: 目标redis的数据库索引。如果要迁移到0号数据库,这里就是0
timeout:迁移的超时时间。
[copy]: 如果添加此选项,则表示迁移之后不删除源键
[replace]:如果添加此选项,migrate不管目标redis是否存在该键都会正常迁移进行数据覆盖。
[KEYS key [key ...]] :如果要迁移多个键那么就将要迁移的键放在这里。
```


>&nbsp;&nbsp;&nbsp;&nbsp;原理如下所示:
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/12.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;需要注意的有以下几点:

```text
1. migrate是个原子操作。只是需要在源redis上执行。
2. migrate命令的数据传输直接从源redis和目标redis上完成的。
3. 目标redis完成restore之后会发送OK给源redis,源redis接收后
会根据migrate对应的选项来决定是否在源redis上删除对应的键。
```

# 遍历键

## 查看所有的键

```redis
    keys *
```

## 带有匹配模式

```redis
key pattern 

pattern:
    *代表匹配任意字符
    .代表匹配一个字符
    []:代表匹配部分字符,例如[1,3]代表匹配1,3。
    \x:用来做转义
```

>&nbsp;&nbsp;&nbsp;&nbsp;如果redis中的键过多,那么执行keys就可能阻塞redis阻塞。一般情况下不要使用keys命令。可以使用scan命令渐进式的遍历所有的键。


## 渐进式遍历


>&nbsp;&nbsp;&nbsp;&nbsp;scan采用渐进式遍历的方式来解决keys命令可能带来的阻塞问题。每次的时间复杂度是O(1)。redis存储键值对实际使用的是hashtable的数据结构。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/13.jpg?raw=true)


```redis
scan cursor [match pattern] [count number]

cursor:是一个游标,第一次从0开始,每次scan遍历都会返回当前游标的值，直到游标值是0,表示遍历结束。

match pattern: 是可选参数,作用是做模式的匹配,和keys的模式匹配很像。

count number:是可选参数,作用是表名每次都要遍历的键个数,默认是10,参数可以适当调大。
```


# 数据库管理

## 切换数据库

```redis
select dbindex

redis默认是16个库,当dbindex是16的时候,将会切换到第一个库.
库与库之间的key可以是相同的.

我们公司只是使用一个库。哈哈哈
```

## 清空数据库

```redis
flushdb:清空当前库

flushall: 清空所有的库 
```

