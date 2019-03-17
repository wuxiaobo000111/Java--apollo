# MySQL  Cluster 概述

>&nbsp;&nbsp;&nbsp;&nbsp;MySQL Cluster技术在分布式系统中为MySQL提供了冗余特性，增强了安全性，可以大大的提高系统的可靠性和数据的有效性。MySQL集群需要一组计算机，每台计算机可以理解为一个节点，这些节点的功能各不相同。MySQL Cluster按照功能来分，可以分为三种节点：管理节点、数据节点和SQL节点。集群中的某台计算机可以是某一个节点，也可以是两种或者三种节点的集合，这些节点组合在一起，为应用提供具有高可靠性、高性能的Cluster数据管理。


>&nbsp;&nbsp;&nbsp;&nbsp;MySQL Cluster简单地讲是一种MySQL集群的技术，是由一组计算机构成，每台计算机可以存放一个或者多个节点，其中包括MySQL服务器，DNB Cluster的数据节点，管理其他节点，以及专门的数据访问程序，这些节点组合在一起，就可以为应用提高可高性能、高可用性和可缩放性的Cluster数据管理；

 

>&nbsp;&nbsp;&nbsp;&nbsp;MySQL Cluster的访问过程大致是这样的，应用通常使用一定的负载均衡算法将对数据访问分散到不同的SQL节点，SQL节点对数据节点进行数据访问并从数据节点返回数据结果，管理节点仅仅只是对SQL节点和数据节点进行配置管理；

 
# 节点

## 管理节点
>&nbsp;&nbsp;&nbsp;&nbsp;管理节点主要是用来对其他的节点进行管理。通常通过配置config.ini文件来配置集群中有多少需要维护的副本、配置每个数据节点上为数据和索引分配多少内存、IP地址、以及在每个数据节点上保存数据的磁盘路径；

> &nbsp;&nbsp;&nbsp;&nbsp;管理节点通常管理Cluster配置文件和Cluster日志。Cluster中的每个节点从管理服务器检索配置信息，并请求确定管理服务器所在位置的方式。如果节点内出现新的事件的时候，节点将这类事件的信息传输到管理服务器，将这类信息写入到Cluster日志中。
> 
>&nbsp;&nbsp;&nbsp;&nbsp;一般在MySQL Cluster体系中至少需要一个管理节点，另外值得注意的是，因为数据节点和SQL节点在启动之前需要读取Cluster的配置信息，所以通常管理节点是最先启动的。
## SQL节点

>&nbsp;&nbsp;&nbsp;&nbsp; SQL节点简单地讲就是mysqld服务器，应用不能直接访问数据节点，只能通过SQL节点访问数据节点来返回数据。任何一个SQL节点都是连接到所有的存储节点的，所以当任何一个存储节点发生故障的时候，SQL节点都可以把请求转移到另一个存储节点执行。通常来讲，SQL节点越多越好，SQL节点越多，分配到每个SQL节点的负载就越小，系统的整体性能就越好；

## 数据节点

>&nbsp;&nbsp;&nbsp;&nbsp;   数据节点用来存放Cluster里面的数据，MySQL Cluster在各个数据节点之间复制数据，任何一个节点发生了故障，始终会有另外的数据节点存储数据。


![最基础的MySQL cluster](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-17/1.png?raw=true "")

>&nbsp;&nbsp;&nbsp;&nbsp;努力了一天，也没有实现上述这个cluster,主要遇到的问题是:

![](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-17/7.jpg?raw=true "")

>&nbsp;&nbsp;&nbsp;&nbsp;基本上试了百度上所有的方法，一直没有解决......



