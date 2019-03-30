# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;可以为redis做基准性能测试。

```redis
-c 
    -c(clients)表示客户端的并发量(默认是50)

-n<requests>

    -n(num)选项代表客户端的请求总数

---redis-benchmark -c 100 -n 20000---表示100客户端同时请求,一共请求20000次 。

-q

    redis-benchmark的requests per senond信息

-r  向redis中插入更多的随机键

---redis-benchmark -c 100 -n 20000 -r 10000---
    -r 10000表示只会对后四位做随机处理
-p
    
    每个请求pipeline的数据量(默认是1)

-k<boolean>

    -k表示客户端是否使用keepalive 1表示使用,0表示不使用,默认值是1.
-t

    对指定命令记性基准测试

--csv 
    
    会将结果按照csv格式输出

```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-29/4.jpg?raw=true)
