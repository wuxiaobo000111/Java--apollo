![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-31/2.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;上图介绍了发布订阅模式的模型。


# 发布订阅命令



```redis
publish channelname message

    向channelname中发送消息

subscribe channelname [channelname]

    订阅多个频道 
unsubscribe channelname[channelname]

    取消订阅

psubscribe pattern [pattern]

    按照模式订阅

punsubscribe pattern [pattern]

    按照模式取消订阅


pubsub channels [pattern]

    查看活跃的频道

pubsub numsub [channel .......]

    查看频道订阅数

pubsub numpat
    查看模式订阅数    
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-31/1.jpg?raw=true)


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-31/3.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;需要注意一下几点

```text
1. 客户端在执行订阅命令之后就进入了订阅状态,只能接受subscribe、psubscribe、unsubscribe、punsubscribe命令。

2.新开启的订阅客户端,无法收到该频道之前的消息,因为redis不会对发布的消息进行持久化。
```