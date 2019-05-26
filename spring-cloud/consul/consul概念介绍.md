>原博客地址：https://blog.csdn.net/love_zngy/article/details/82216696
# 什么是consul

>&nbsp;&nbsp;&nbsp;&nbsp;Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其它分布式服务注册与发现的方案，Consul 的方案更“一站式”，内置了服务注册与发现框 架、分布一致性协议实现、健康检查、Key/Value 存储、多数据中心方案，不再需要依赖其它工具（比如 ZooKeeper 等）。使用起来也较 为简单。Consul 使用 Go 语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与 Docker 等轻量级容器可无缝配合。

# consul能做什么

```text

1.服务发现。有了consul,服务可以通过DNS或者HTTP直接找到它所依赖的服务。
2.健康检查。consul提供了健康检查的机制,从简单的服务端是否返回200的响应代码到较为复杂的内存使用率是否低于90%。
3.K/V存储。应用程序可以根据需要使用consul的Key/Value存储。Consul提供了简单易用的http接口来满足用户的动态配置、
特征标记、协调、leader选举等需求。
4.多数据中心。Consul原生支持多数据中心。这意味着用户不用为了多数据中心自己做抽象。

```

# 和其他服务发现做对比

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/37.jpg?raw=true)
