# Zookeeper命令

<a name="dL64J"></a>
# connect
连接zk服务端，与close命令配合使用可以连接或者断开zk服务端。如connect 127.0.0.1:2181

<a name="VZpi3"></a>
# get
获取节点信息，注意节点的路径皆为绝对路径，也就是说必要要从/（根路径）开始。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/382109/1561026638389-b9dd57e3-2402-4ea4-971c-95776cd5f8a8.png#align=left&display=inline&height=367&name=image.png&originHeight=404&originWidth=981&size=106317&status=done&width=891.8181624885437)

```
hello world为节点数据信息
cZxid节点创建时的zxid
ctime节点创建时间
mZxid节点最近一次更新时的zxid
mtime节点最近一次更新的时间
cversion子节点数据更新次数
dataVersion本节点数据更新次数
aclVersion节点ACL(授权信息)的更新次数
ephemeralOwner如果该节点为临时节点,ephemeralOwner值表示与该节点绑定的session id.
如果该节点不是临时节点,ephemeralOwner值为0
dataLength节点数据长度，本例中为hello world的长度
numChildren子节点个数
```

<a name="DdL7E"></a>
# ls
获取路径下的节点信息，注意此路径为绝对路径，类似于linux的ls命令。如ls /zookeeper

<a name="dr3MY"></a>
# set
设置节点的数据。如set /zookeeper "hello world"

<a name="04kCk"></a>
# rmr
删除节点命令，此命令与delete命令不同的是delete不可删除有子节点的节点，但是rmr命令可以删除，注意路径为绝对路径。如rmr /zookeeper/znode

<a name="phU4c"></a>
# setquota
设置子节点个数和数据长度配额。如setquota –n 4 /zookeeper/node 设置/zookeeper/node子节点个数最大为4，setquota –b 100 /zookeeper/node 设置/zookeeper/node节点长度最大为100

<a name="iUtjq"></a>
# listquota
显示配额。

```
listquota /zookeeper
absolute path is/zookeeper/quota/zookeeper/zookeeper_limits
Output quota for /zookeepercount=2,bytes=-1

解释：/zookeeper节点个数限额为2，长度无限额。
```


<a name="ECJ6o"></a>
# quit
退出。

<a name="vX2AP"></a>
# printwatches
设置和显示监视状态，on或者off。如printwatches on

<a name="ItJ2d"></a>
# create
创建节点，其中-s为顺序充点，-e临时节点。如create /zookeeper/node1"test_create" world:anyone:cdrwa。其中acl处，请参见getAcl和setAcl命令。

<a name="SeQLn"></a>
# stat
查看节点状态信息。如stat /

<a name="MhsFo"></a>
# close
> 断开客户端与服务端的连接。
> 

<a name="8Au5O"></a>
# [](#ls2命令)ls2
> ls2为ls命令的扩展，比ls命令多输出本节点信息。
> 

<a name="zRbXe"></a>
# [](#history命令)history
> 列出最近的历史命令。
> 

<a name="J5P78"></a>
# [](#setacl命令)setAcl
设置节点Acl。此处重点说一下acl，acl由大部分组成：1为scheme，2为user，3为permission，一般情况下表示为scheme:id:permissions。其中scheme和id是相关的，下面将scheme和id一起说明。<br />scheme和id

  - world: 它下面只有一个id, 叫anyone, world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的
  - auth: 它不需要id, 只要是通过authentication的user都有权限（zookeeper支持通过kerberos来进行authencation, 也支持username/password形式的authentication)
  - digest: 它对应的id为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication
  - ip: 它对应的id为客户机的IP地址，设置的时候可以设置一个ip段，比如ip:192.168.1.0/16, 表示匹配前16个bit的IP段
  - super: 在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(cdrwa)

permissions

  - CREATE(c): 创建权限，可以在在当前node下创建child node
  - DELETE(d): 删除权限，可以删除当前的node
  - READ(r): 读权限，可以获取当前node的数据，可以list当前node所有的child nodes
  - WRITE(w): 写权限，可以向当前node写数据
  - ADMIN(a): 管理权限，可以设置当前node的permission

综上，一个简单使用setAcl命令，则可以为：<br />setAcl /zookeeper/node1 world:anyone:cdrw<br />

<a name="Cs7hV"></a>
# [](#getacl命令)getAcl
获取节点Acl。如getAcl /zookeeper/node1 'world,'anyone: cdrwa 注：可参见setAcl命令。<br />

<a name="3i9V8"></a>
# [](#sync命令)sync
强制同步。如sync /zookeeper,由于请求在半数以上的zk server上生效就表示此请求生效，那么就会有一些zk server上的数据是旧的。sync命令就是强制同步所有的更新操作。<br />
<br />

<a name="040Fh"></a>
# [](#redo命令)redo
再次执行某命令。如redo 10.其中10为命令ID，需与history配合使用。<br />

<a name="qVQs9"></a>
# [](#addauth命令)addauth
节点认证。如addauth digest username:password，可参见setAcl命令digest处。
```
使用方法：
一、通过setAcl设置用户名和密码
setAcl pathdigest:username:base64(sha1(password)):crwda
二、认证
addauth digest username:password
```
<a name="1XyuC"></a>
### 
<a name="JWGyX"></a>
# delete
删除节点。如delete /zknode1
<a name="bibuH"></a>
# 
<br />
<a name="mBtnQ"></a>
# 
