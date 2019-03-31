
# RDB和AOF的配置
# rdb

## 概述
>&nbsp;&nbsp;&nbsp;&nbsp;RDB是在某个时间点将数据写入一个临时文件，持久化结束后，用这个临时文件替换上次持久化的文件，达到数据恢复。

>&nbsp;&nbsp;&nbsp;&nbsp;优点：使用单独子进程来进行持久化，主进程不会进行任何IO操作，保证了redis的高性能

>&nbsp;&nbsp;&nbsp;&nbsp; 缺点：RDB是间隔一段时间进行持久化，如果持久化之间redis发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候

## 配置信息
```properties
#dbfilename：持久化数据存储在本地的文件
dbfilename dump.rdb
#dir：持久化数据存储在本地的路径，如果是在/redis/redis-3.0.6/src下启动的redis-cli，则数据会存储在当前src目录下
dir ./
##snapshot触发的时机，save <seconds> <changes>  
##如下为900秒后，至少有一个变更操作，才会snapshot  
##对于此值的设置，需要谨慎，评估系统的变更操作密集程度  
##可以通过“save “””来关闭snapshot功能  
#save时间，以下分别表示更改了1个key时间隔900s进行持久化存储；更改了10个key300s进行存储；更改10000个key60s进行存储。
save 900 1
save 300 10
save 60 10000
##当snapshot时出现错误无法继续时，是否阻塞客户端“变更操作”，“错误”可能因为磁盘已满/磁盘故障/OS级别异常等  
stop-writes-on-bgsave-error yes  
##是否启用rdb文件压缩，默认为“yes”，压缩往往意味着“额外的cpu消耗”，同时也意味这较小的文件尺寸以及较短的网络传输时间  
rdbcompression yes  
```

>&nbsp;&nbsp;&nbsp;&nbsp;snapshot触发的时机，是有“间隔时间”和“变更次数”共同决定，同时符合2个条件才会触发snapshot,否则“变更次数”会被继续累加到下一个“间隔时间”上。snapshot过程中并不阻塞客户端请求。snapshot首先将数据写入临时文件，当成功结束后，将临时文件重名为dump.rdb。


## RDB触发机制

```text
1.save命令:阻塞当前redis服务器,直到RDB过程完成为止,如果redis数据较多,可能造成redis进程的长时间阻塞。

2.bgsave: redis执行fork创建子进程,RDB持久化过程由这个子进程负责,完成之后结束。
```


## 流程说明

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-31/4.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;执行步骤如下所示

```text

1.执行bgsave命令,Redis父进程判断当前是否存在正在执行的子进程,如RDB/AOF子进程,如果存在则直接返回。

2.父进程执行fork操作创建子进程,fork一个子进程会阻塞父进程。

3.父进程fork完成之后,bgsave命令返回一个信息给父进程,这个时候父进程并不会阻塞。

4.子进程创建RDB文件,根据父进程内存生成临时快照,完成之后对原有的文件进行原子替换。

5.子进程发送信号给父进程表示完成,父进程更新统计信息。

```

 
## 使用RDB恢复数据：
 >&nbsp;&nbsp;&nbsp;&nbsp; RDB文件的载入工作是在服务器启动时自动执行的，并没有专门的命令。但是由于AOF的优先级更高，因此当AOF开启时，Redis会优先载入AOF文件来恢复数据；只有当AOF关闭时，才会在Redis服务器启动时检测RDB文件，并自动载入。服务器载入RDB文件期间处于阻塞状态，直到载入完成为止。
 
 >&nbsp;&nbsp;&nbsp;&nbsp;Redis载入RDB文件时，会对RDB文件进行校验，如果文件损坏，则日志中会打印错误，Redis启动失败。


# AOF
>&nbsp;&nbsp;&nbsp;&nbsp;Append-only file，将“操作 + 数据”以格式化指令的方式追加到操作日志文件的尾部，在append操作返回后(已经写入到文件或者即将写入)，才进行实际的数据变更，“日志文件”保存了历史所有的操作过程；当server需要数据恢复时，可以直接replay此日志文件，即可还原所有的操作过程。AOF相对可靠，它和mysql中bin.log、apache.log、zookeeper中txn-log简直异曲同工。AOF文件内容是字符串，非常容易阅读和解析。

>&nbsp;&nbsp;&nbsp;&nbsp;优点：可以保持更高的数据完整性，如果设置追加file的时间是1s，如果redis发生故障，最多会丢失1s的数据；且如果日志写入不完整支持redis-check-aof来进行日志修复；AOF文件没被rewrite之前（文件过大时会对命令进行合并重写），可以删除其中的某些命令（比如误操作的flushall）。

>&nbsp;&nbsp;&nbsp;&nbsp;缺点：AOF文件比RDB文件大，且恢复速度慢。

>&nbsp;&nbsp;&nbsp;&nbsp;我们可以简单的认为AOF就是日志文件，此文件只会记录“变更操作”(例如：set/del等)，如果server中持续的大量变更操作，将会导致AOF文件非常的庞大，意味着server失效后，数据恢复的过程将会很长；事实上，一条数据经过多次变更，将会产生多条AOF记录，其实只要保存当前的状态，历史的操作记录是可以抛弃的；因为AOF持久化模式还伴生了“AOF rewrite”。

> &nbsp;&nbsp;&nbsp;&nbsp;AOF的特性决定了它相对比较安全，如果你期望数据更少的丢失，那么可以采用AOF模式。如果AOF文件正在被写入时突然server失效，有可能导致文件的最后一次记录是不完整，你可以通过手工或者程序的方式去检测并修正不完整的记录，以便通过aof文件恢复能够正常；同时需要提醒，如果你的redis持久化手段中有aof，那么在server故障失效后再次启动前，需要检测aof文件的完整性。

>&nbsp;&nbsp;&nbsp;&nbsp;AOF是文件操作，对于变更操作比较密集的server，那么必将造成磁盘IO的负荷加重；此外linux对文件操作采取了“延迟写入”手段，即并非每次write操作都会触发实际磁盘操作，而是进入了buffer中，当buffer数据达到阀值时触发实际写入(也有其他时机)，这是linux对文件系统的优化，但是这却有可能带来隐患，如果buffer没有刷新到磁盘，此时物理机器失效(比如断电)，那么有可能导致最后一条或者多条aof记录的丢失。通过上述配置文件，可以得知redis提供了3中aof记录同步选项：


>&nbsp;&nbsp;&nbsp;&nbsp;其实，我们可以选择的太少，everysec是最佳的选择。如果你非常在意每个数据都极其可靠，建议你选择一款“关系性数据库”吧。AOF文件会不断增大，它的大小直接影响“故障恢复”的时间,而且AOF文件中历史操作是可以丢弃的。**AOF rewrite操作就是“压缩”AOF文件的过程，当然redis并没有采用“基于原aof文件”来重写的方式，而是采取了类似snapshot的方式：基于copy-on-write，全量遍历内存中数据，然后逐个序列到aof文件中。因此AOF rewrite能够正确反应当前内存数据的状态，这正是我们所需要的；**rewrite过程中，对于新的变更操作将仍然被写入到原AOF文件中，同时这些新的变更操作也会被redis收集起来(buffer，copy-on-write方式下，最极端的可能是所有的key都在此期间被修改，将会耗费2倍内存)，当内存数据被全部写入到新的aof文件之后，收集的新的变更操作也将会一并追加到新的aof文件中，此后将会重命名新的aof文件为appendonly.aof,此后所有的操作都将被写入新的aof文件。如果在rewrite过程中，出现故障，将不会影响原AOF文件的正常工作，只有当rewrite完成之后才会切换文件，因为rewrite过程是比较可靠的。


>&nbsp;&nbsp;&nbsp;&nbsp;可以通过配置文件来指定它们中的一种，或者同时使用它们(不建议同时使用)，或者全部禁用，在架构良好的环境中，master通常使用AOF，slave使用snapshot，主要原因是master需要首先确保数据完整性，它作为数据备份的第一选择；slave提供只读服务(目前slave只能提供读取服务)，它的主要目的就是快速响应客户端read请求；但是如果你的redis运行在网络稳定性差/物理环境糟糕情况下，建议你master和slave均采取AOF，这个在master和slave角色切换时，可以减少“人工数据备份”/“人工引导数据恢复”的时间成本；如果你的环境一切非常良好，且服务需要接收密集性的write操作，那么建议master采取snapshot，而slave采用AOF。


## AOF流程

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-31/5.jpg?raw=true)


```text
1.所有的写入命令都会追加到aof_buf(缓冲区)中。
2.AOF缓冲区根据对应的策略向硬盘中做同步操作。
3.随着AOF文件的增大,需要定期对AOF文件进行重写,达到压缩的目的。
4.当Redis服务器重启的时候,可以加载AOF文件进行数据恢复。
```

### 命令写入

>&nbsp;&nbsp;&nbsp;&nbsp;AOF命令的写入是使用文本协议格式。为什么要首先写入到缓冲区,是因为Redis是单线程的,如果每次都追加到硬盘中,那么性能就取决于当前硬盘的负载。


### 文件同步

>&nbsp;&nbsp;&nbsp;&nbsp;由appendfsync进行控制。
```text
always：每一条aof记录都立即同步到文件，这是最安全的方式，也以为更多的磁盘操作和阻塞延迟，
是IO开支较大。

everysec：每秒同步一次，性能和安全都比较中庸的方式，也是redis推荐的方式。如果遇到物理服务器故障，
有可能导致最近一秒内aof记录丢失(可能为部分丢失)。

no：redis并不直接调用文件同步，而是交给操作系统来处理，操作系统可以根据buffer填充情况/通道空闲时
间等择机触发同步；这是一种普通的文件操作方式。性能较好，在物理服务器故障时，数据丢失量会因OS配置有关。

```

### 重写机制

>&nbsp;&nbsp;&nbsp;&nbsp;AOF重写过程可以手动触发也可以自动触发。
```text
1.手动触发:直接调用bgrewriteaof命令
2.自动触发:根据auto-aof-rewrite-min-size和auto-aof-rewrite-percentage参数来决定自动触发。

auto-aof-rewrite-min-size表示运行AOF重写文件的最小体积
auto-aof-rewrite-percentage代表当前AOF文件空间和上次从写后AOF文件空间的比值。

```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-31/6.jpg?raw=true)

```text
1. 执行AOF重写请求。 如果当前进程正在执行AOF重写,请求执行。
2. 父进程执行fork创建子进程,开销等同于bgsave过程。
3. 
    主进程fork操作完成后,继续相应其他命令。所有修改命令依然写入AOF缓冲区并根绝appendfsync策略同步到硬盘,
    保证原有AOF机制正确性。

    fork操作运用写时复制技术,子进程只能共享fork操作时候的内存数据。由于父进程依然响应命令,redis使用“AOF
    重写缓冲区”保存这部分西数据,防止新AOF文件生成期间丢失这部分数据。

4. 子进程依据内存快照,按照命令合并规则写入新的AOF文件。

5.

    新AOF文件写入完成之后,子进程发送信号给父进程,父进程统计信息。

    父进程把AOF重写缓冲区的数据写入到新的AOF文件。

    使用新AOF文件替换老文件,完成AOF重写。

```

# 重启加载

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-31/7.jpg?raw=true)
