# MySQL慢查询日志

>&nbsp;&nbsp;&nbsp;&nbsp;MySQL有一种日志，叫做慢查询日志，主要就是用来记录一些耗时的查询操作。通过这个日志我们就可以分析出哪些的操作是影响性能的，我们需要对其进行一些优化措施。

# 如何查看是否开启

```sql
show variables  like '%quer%';
```

![](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-17/8.jpg?raw=true "")


>&nbsp;&nbsp;&nbsp;&nbsp;我的这个是没有开启的，如果想要开启，则就需要指定如下配置:
```properties
#指定开启慢查询日志
slow-query-log=1
#指定慢查询日志的路径
slow_query_log_file="mysql-slow.log"
#指定查询时间大于多少的才进行记录，但是是毫秒，也就是操作大于 10ms 的操作都会被记录。
long_query_time=10
```
>&nbsp;&nbsp;&nbsp;&nbsp;修改过后可以看出如下所示：

![](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-17/9.jpg?raw=true "")
