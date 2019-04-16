# eureka常用配置项

```
eureka.client.allow-redirects:是否允许重定向Eureka客户端请求到其他或者备份服务器，默认为fasle

eureka.client.eureka-connection-idle-timeout-seconds:HTTP连接到eureka服务器可以在关闭之前保持空闲的时间(几秒钟)。

eureka.client.eureka-server-connect-timeout-seconds:表示连接Eureka服务器，等待多长时间算超时

eureka.client.eureka-server-port： Eureka Server端口

eureka.client.eureka-server-d-n-s-name：获取要查询的DNS名称以获得eureka服务器的列表。

eureka.client.eureka-server-read-timeout-seconds：示在从eureka服务器读取数据之前需要等待多长时间(以秒为单位)

eureka.client.eureka-server-total-connections：从eureka客户端到所有eureka服务器的所允许连接总数。

eureka.client.eureka-server-total-connections-per-host：设置每一个主机所允许的到Eureka Server连接的数量

eureka.client.fetch-registry： 是否允许客户端向Eureka 注册表获取信息，一般服务器为设置为false,客户端设置为true

eureka.client.register-with-eureka：是否允许向Eureka Server注册信息，默认true,如果是服务器端，应该设置为false

eureka.client.fetch-remote-regions-registry：逗号分隔的区域列表，用于获取eureka注册信息

eureka.client.g-zip-content：从服务器端获取数据是否需要压缩

eureka.client.prefer-same-zone-eureka： 是否优先使选择相同Zone的实例,默认为true

eureka.client.registry-fetch-interval-seconds:多长时间从Eureka Server注册表获取一次数据，默认30s

eureka.client.service-url:可用区域映射，列出完全合格的url与eureka服务器通信。每个值可以是一个URL，也可以是一个逗号分隔的替代位置列表。

eureka.dashboard.enabled： 是否启用Eureka首页，默认为true

eureka.dashboard.path： 默认为/

eureka.instance.appname：在eureka注册的应用程序的名称。

eureka.instance.app-group-name：在eureka注册的应用程序的组名称

eureka.instance.health-check-url： 健康检查绝对路径

eureka.instance.health-check-url-path：健康检查相对路径

eureka.instance.hostname：设置主机名

eureka.instance.instance-id：设置注册实例的id

eureka.instance.lease-expiration-duration-in-seconds：设置多长时间意味着租约到期，默认90

eureka.instance.lease-renewal-interval-in-seconds:表示Eureka客户端需要发送心跳到eureka服务器的频率(以秒为单位)，以表明它仍然存在。指定的期间内如果没有收到心跳leaseExpirationDurationInSeconds

eureka.instance.metadata-map:可以设置元数据

eureka.instance.prefer-ip-address： 实例名以IP，但是建议hostname,默认为false
```