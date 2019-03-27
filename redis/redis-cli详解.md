# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这里说明一下redisc-cli的具体命令

```redis 
-r times: 代表这个命令将会times多次
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/18.jpg?raw=true)


```redis
-i seconds :每隔几秒就会执行一次(必须和-r一起使用,同时不支持毫秒级别)
```
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/19.jpg?raw=true)


```redis
-x :从标准输入读取数据作为redis-cli的最后一个参数。
```
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/20.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;演示将输入world,设置key为hello的value是world。

```redis
-c: 连接redis cluster 节点的时候需要使用
```

```redis
-a: 如果配置了密码,可以通过-a直接输入密码
```

```redis
--scan和--pattern : 用于扫描指定模式的键,相当于使用scan命令
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/21.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;匹配所有的key是以name11111开头。


```redis
--slave: 把当前客户端模拟成为当前redis节点的从节点,可以用来获取当前redis节点的更新操作。
```

```redis
--rdb:请求redis实例生成并发送RDB持久化文件,保存到本地。

```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/22.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;将rdb文件写入到/usr/local/redis/bin/dump1.rdb文件中。

```redis
--pipe:将命令封装成为Redis通信协议定义的数据格式,批量发送给redis执行。
```


```redis
--bigkeys:使用scan命令对redis进行采样,从中找到内存占用比较大的键值对。这些键值对可能是系统的瓶颈。
```

```redis

--eval:执行指定的Lua脚本。
```

```redis

--stat: 可以实时获取redis的重要统计信息
    
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/23.jpg?raw=true)

```redis
--raw 返回格式化之后的结果。
--no-raw 要求命令返回的是原始数据
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/24.jpg?raw=true)
