
# sentinel monitor

```text
sentinel monitor <master-name> <ip> <port> <quorum>

master-name是一个别名。
quorum表示判定主节点不可达所需要的票数。

这里没有配置从节点的信息,是因为从节点的信息可以通过主节点的信息获取到。
```

# sentinel down-after-milliseconds 

```text
sentinel down-after-milliseconds <master-name> <times>

1.Sentinel节点都要通过定期发送ping命令来判断redis数据节点和其他Sentinel节点是否可达。如果超过了
down-after-milliseconds,则判断节点不可达。

```


# sentinel parallel-syncs 

```text
sentinel parallel-syncs  <master-name> <nums>

当发生故障转移的时候,原来的从节点就需要向新的master发起复制的操作。这个sentinel parallel-syncs就是说明向新的
master发起复制操作的slave的个数。设置为1表示是从节点轮询发起复制。
```


# sentinel failover-timeout

```text
sentinel failover-timeout <master-name> <times>

故障转移的超时时间,具体作用如下

    1.如果Sentinel对一个主节点的故障转移失败,那么下次再次对该主节点做故障转移的起始时间是failover-timeout的2倍。

    2.在从节点选出新的主节点的时候,如果选出来的从节点执行slave no one一直失败,如果超过failover-time时间
    转移故障失败。

    3. 如果选出新的master成功,还会执行info命令确认是否晋升成功,如果超过failover-time时间，则故障转移失败。

    4. 如果复制新的master超时,故障转移失败。

```

# sentinel auth-pass

```text
sentinel auth-pass <master-name> <password>

Sentinel添加主节点的密码
```


# sentinel notification-script

```text
sentinel notification-script <master-name> <script-path>

Sentinel在故障转移期间，当发何时能一些警告的时候，会触发一些脚本,这里指定了脚本的路径。
```



