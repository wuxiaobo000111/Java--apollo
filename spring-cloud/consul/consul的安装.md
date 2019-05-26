# 下载

>&nbsp;&nbsp;&nbsp;&nbsp;下载地址是https://www.consul.io/downloads.html。然后上传到自己的linux服务器。


# 启动命令

>&nbsp;&nbsp;&nbsp;&nbsp;./consul agent -dev -ui -client 192.168.88.128 -dev表示是启动的开发环境下的一个Server节点。-ui表示开启consul的UI页面。 -clent 192.168.88.128 表示是可以通过这个地址访问consul。<font color =red>注意:这里其实有一个很大的坑,对于初学者来说可能是一个问题就是说如果指定agent -dev 也是只允许当前ip注册，但这个是本地开发时用的，正式服务器往往不用。所以如下的配置是错误的:</font>

```yml
server:
  port: 8109
spring:
  application:
    name: consul-server
  cloud:
    consul:
      discovery:
        service-name: consul-server
        ip-address: 192.168.88.128
#        consul所在的ip地址
      host: 192.168.88.128
#     访问端口
      port: 8500

```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/38.jpg?raw=true)




# consule常用的命令


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/39.jpg?raw=true)

# consul agent 命令的常用选项

```text

-data-dir 作用：指定agent储存状态的数据目录
          这是所有agent都必须的.对于server尤其重要，因为他们必须持久化集群的状态

-config-dir 作用：指定service的配置文件和检查定义所在的位置
          通常会指定为”某一个路径/consul.d”（通常情况下，.d表示一系列配置文件存放的目录）

-config-file 作用：指定一个要装载的配置文件
          该选项可以配置多次，进而配置多个配置文件（后边的会合并前边的，相同的值覆盖）

-dev 作用：创建一个开发环境下的server节点
          该参数配置下，不会有任何持久化操作，即不会有任何数据写入到磁盘.这种模式不能用于生产环境（因为第二条）

-bootstrap-expect  作用：该命令通知consul server我们现在准备加入的server节点个数，该参数是为了延迟日志复制的启动
直到我们指定数量的server节点成功的加入后启动。

-node 作用：指定节点在集群中的名称
        该名称在集群中必须是唯一的（默认采用机器的host）
        推荐：直接采用机器的IP

-bind 作用：指明节点的IP地址
        有时候不指定绑定IP，会报Failed to get advertise address: Multiple private IPs found. 
        Please configure one. 的异常

-server 作用：指定节点为server
        每个数据中心（DC）的server数推荐至少为1，至多为5所有的server都采用raft一致性算法来确保事务的一致性
        和线性化，事务修改了集群的状态，且集群的状态保存在每一台server上保证可用性server也是与其他DC交互的门面
        （gateway）


-client 作用：指定节点为client，指定客户端接口的绑定地址，包括：HTTP、DNS、RPC 默认是127.0.0.1，只允许回环
接口访问若不指定为-server，其实就是-client

-join 作用：将节点加入到集群

-datacenter（老版本叫-dc，-dc已经失效）

```

