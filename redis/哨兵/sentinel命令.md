
# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这里讲述一下Sentinel的命令。

# sentinel masters

>&nbsp;&nbsp;&nbsp;&nbsp;作用:展示所有被监控的主节点以及相关的统计信息

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-14/14.jpg?raw=true)


# sentinel master
```
sentinel master  <master-name>

展示指定的master-name的主节点状态以及相关的统计信息。
其中master-name就是在配置哨兵的时候使用的别名。

```

# sentinel slaves

```
sentinel slaves <master-name>
展示指定master-name的从节点状态以及相关的统计信息。
```

# sentinel sentinels 

```
sentinel sentinels <master-name>
展示指定master-name节点的sentinel节点的集合。
```

# sentinel get-master-addr-by-name 
```
sentinel get-master-addr-by-name <master-name>
获取主节点的IP地址和端口

```

# sentinel reset
```
sentinel reset <pattern>
对复合<pattern>主节点的配置进行重置,包含清楚主节点的相关状态,重新发送从节点和Sentinel节点。

```

# sentinel failover 

```
sentinel failover <master name>
对指定master-name主节点进行强制故障转移,当故障转移完成之后,其他sentinel节点按照故障转移的结果更新自身配置。

```


# sentinel ckquorum 
```
sentinel ckquorum <master-name>
检测当前可达的sentinel节点总数是否达到了<quorum>的个数。
```


# sentinel flushconfig 

>&nbsp;&nbsp;&nbsp;&nbsp;将sentinel节点的配置强制刷新到磁盘上去。



# sentinel remove
```
sentinel remove <master-name>

取消当前sentinel节点对于指定<master-name>主节点的监控
```

# sentinel monitor

```
sentinel monitor <master-name> <ip> <port> <quorum>

通过命令的形式完成sentinel对主节点的监控
```

# sentinel set

```
sentinel set <master-name>
动态修改sentinel节点的配置信息
```


# sentinel is-master-down-by-addr

>&nbsp;&nbsp;&nbsp;&nbsp;sentinel节点之间用来交换对主节点是否下线的判断。


