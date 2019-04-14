![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/1.jpg?raw=true)

# 启动主节点

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/2.jpg?raw=true)


# 启动两个从节点

>&nbsp;&nbsp;&nbsp;&nbsp;需要注意的是在启动两个从节点之前需要先修改端口

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/3.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/4.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/5.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/6.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/7.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;这个时候主从其实已经搭建好了,其中6379是master节点。6380、5381是slave节点。


# Sentinel的启动

## 启动sentinel1

>&nbsp;&nbsp;&nbsp;&nbsp;修改的配置参数如下所示:
``` properties
port 26379
logfile "26379.log"
daemonize yes
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000

# sentinel monitor mymaster 127.0.0.1 6379 2表示sentinel监控127.0.0.1:6379这个节点。2表示主节点失败了至少
需要2个Sentinel节点同意。
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/8.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;验证是否成功。同时启动其他两个节点。只是需要修改上面的端口和日志目录不同就行。



![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/10.jpg?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/11.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;判断是否成功的标志:
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/13.jpg?raw=true)
