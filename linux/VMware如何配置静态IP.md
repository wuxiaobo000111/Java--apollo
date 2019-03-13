
### 概述

>这篇文章教你如何配置VMware虚拟机中的静态IP。

#### 设置虚拟机的网络

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313211146376.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313211259986.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313211345298.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)

>然后点击确定就行了。

#### 修改VMnet8的IPV4的相关配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313211450534.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313211909247.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)


#### 设置虚拟机的IP

>vim /etc/sysconfig/network-scripts/ifcfg-ens33 


```text
YPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="1718c2cf-3d7a-4198-af6d-a77835737e19"
DEVICE="ens33"
ONBOOT="yes"
IPADDR=192.168.192.130 #这个IP地址和网关一个网段即可，192.168.192.*
PREFIX=24
GATEWAY=192.168.192.128 #网关，和我们第二步配置中的相同
NETMASK=255.255.255.0 #子网掩码，和我们第二步配置中的相同                                                         
```

>然后更新配置： service network restart


![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313212249789.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)



#### 结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190313212311527.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)


#### 备注
原始博客地址:https://www.cnblogs.com/wangmingshun/p/7708688.html