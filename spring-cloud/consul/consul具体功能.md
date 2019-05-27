>一部分信息来自于《重新定义Spring Cloud实战一书》。这是一本特别好的书。希望大家有时间了可以阅读一波
# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;consul-discovery在功能上分成为服务注册和服务发现。


## 服务注册

>&nbsp;&nbsp;&nbsp;&nbsp;服务启动时,会通过ConsulServiceRegistry.register()向Consul注册自身的服务。注册时,会告诉Consul以下信息。

```text
1.ID,服务ID,默认是服务名-端口号。
2.Name。服务名,默认是系统名称。
3.Tags。给服务打的标签,默认是[secure-false]。
4.Address。服务地址,默认是本机IP。
5.Port。服务端口,默认是服务的web端口。
6.Check.健康检查信息,包括Interval (健康检查间隔)和HTTP (健康检查地址)。
```

>&nbsp;&nbsp;&nbsp;&nbsp;一般情况下,我们不需要显式提供上述信息, spring-cloud-consul会有默认值。但是在一些特殊业务场景中,可能就需要定制上述服务了。consul discovery的常见配置如下所示



![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/42.jpg?raw=true)