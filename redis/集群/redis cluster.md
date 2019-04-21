# 集群功能的限制

```text
1. key批量操作支持有限制,如mset,mget,目前只支持具有相同slot值的key执行批量操作,对于映射为不同slot的key由
于执行mget,mset命令等操作可能存在于多个节点所以不支持。


2. key事务操作支持有限,只支持key在同一个节点上的事务操作,当多个key分布在不同的节点上的时候就无法使用事务功能。

3.key做为数据分区的最小粒度,因此不能将一个大的键值对象hash,list映射到不同的节点。

4.不支持多数据库空间,只能使用一个数据库db0.

5.复制节点只能从主节点复制到从节点。

```


# 集群搭建

## 准备配置文件

>&nbsp;&nbsp;&nbsp;&nbsp;这里使用一台虚拟机,在这台虚拟机中开六个redis节点,模拟redis的集群实现,六台虚拟机做到三主三从。具体节点如下所示。



![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/13.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;新建如下目录,以端口号建立目录,一个redis节点对应一个目录.


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/14.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;编辑redis.conf目录,对应的文件如下所示,不过是每一份的端口号修改一下即可。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/15.png?raw=true)
```
bind 192.168.127.130  //绑定服务器IP地址

port 7000  //绑定端口号，必须修改，以此来区分Redis实例

daemonize yes  //后台运行

pidfile /var/run/redis-7000.pid  //修改pid进程文件名，以端口号命名

logfile /root/application/program/redis-cluster/7000/redis.log  //修改日志文件名称，以端口号为目录来区分

dir /root/application/program/redis-cluster/7000/  //修改数据文件存放地址，以端口号为目录名来区分

cluster-enabled yes  //启用集群

cluster-config-file nodes-7000.conf  //配置每个节点的配置文件，同样以端口号为名称

cluster-node-timeout 15000  //配置集群节点的超时时间，可改可不改

appendonly yes  //启动AOF增量持久化策略

appendfsync always  //发生改变就记录日志
```


>&nbsp;&nbsp;&nbsp;&nbsp;然后分别启动着六个节点:


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/16.jpg?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/17.jpg?raw=true)


>这里说一下,这里之后的redis操作可以使用redis-trib.rb直接命令启动,也可以使用命令行启动。在使用redis-trib.rb的过程中出现了很多的问题,这里只是介绍一下使用命令行启动。对于redis-trib.rb可以之后的时间再进行补充。


## 节点握手

>&nbsp;&nbsp;&nbsp;&nbsp;到目前为止,已经启动了六个节点了,但是这六个节点还没有形成一个集群,下一步需要做的是节点握手。由客户端命令发起:cluster meet [ip][port]。因为我这里是配置过了,所以才会出现如下的结果。但是这个时候的集群还是处于下线的状态。当集群完成了节点握手,但是还没有分配哈希槽。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/18.jpg?raw=true)

## 分配哈希槽

>&nbsp;&nbsp;&nbsp;&nbsp;自己写了一个垃圾脚本用来分配哈希槽。哈哈哈哈。
```sh
node1：
#!/bin/bash
n=0
for ((i=n;i<=5461;i++))
do
   /usr/local/redis/bin/redis-cli -h 127.0.0.1  -p 7000 -a dxy CLUSTER ADDSLOTS $i
done

node2：
#!/bin/bash
n=5462
for ((i=n;i<=10922;i++))
do
   /usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 7001 -a dxy CLUSTER ADDSLOTS $i
done

node3：
#!/bin/bash
n=10923
for ((i=n;i<=16383;i++))
do
   /usr/local/redis/bin/redis-cli -h 127.0.0.1  -p 7002 -a dxy CLUSTER ADDSLOTS $i
done
        
```


>&nbsp;&nbsp;&nbsp;&nbsp;然后去操作redis就行了。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/19.jpg?raw=true)

## 主从配置

>&nbsp;&nbsp;&nbsp;&nbsp;上述操作只是使用了其中的三个节点,然后去分配哈希槽。如果实现主从,就需要使用命令实现主从。 
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-17/21.jpg?raw=true)


# 总结

>&nbsp;&nbsp;&nbsp;&nbsp;我们可以看到如果自己配置实现redis的集群的的确确是超级复杂,当节点挂掉的时候或者是添加节点的时候可能需要自己编写脚本程序了。还是推荐使用redis-trib.rb工具。




