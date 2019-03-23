
# Zset类型

>&nbsp;&nbsp;&nbsp;&nbsp;有序集合和集合一样，都可以包含任意数量的、各不相同的元素（ element），不同于集合的是，有序集合的每个元素都关联着一个浮点数格式的分 值（score），并且有序集合会按照分 值，以从小到大的顺序来排列有序集合中的各个元素。

>&nbsp;&nbsp;&nbsp;&nbsp;虽然有序集合中的每个元素都必 须是各不相同的，但元素的分 值并没有这一限制，换句话来说，两个不同元素的分值可以是相同的。

# 基本操作
```
ZADD key score element [[score element] [score element] ...]
    按照给定的分值和元素，将任意数量的元素添加到有序集合里面，命令的返回 值为成功添加的元素数量。
    复杂度 O(M*log(N))，其中 N 为有序集合已有的元素数量， M 为成功添加的新元素数量。
ZREM key element [element ...]
    从有序集合中删除指定的元素，以及这些元素关联的分值，命令返回被成功删除的元素数量。
    复杂度 O(M*log(N))，其中 N 为有序集合已有的元素数量， M 为成功删除的元素数量。
ZSCORE key element
    返回有序集合中，指定元素的分值。 
    复杂度为 O(1) 。
ZINCRBY key increment element
    为有序集合指定元素的分 值加上增量 increment ，命令返回执行操作之后，元素的分 值。
    没有相应的 ZDECRBY 命令，但可以通过将 increment 设置为负数来减少分值。
    复杂度为 O(log(N)) 。
ZCARD key
    返回有序集合包含的元素数量（基数）。复 杂度为 O(1) 。
ZRANK key element
    返回指定元素在有序集合中的排名，其中 排名按照元素的分值从小到大计算。
    复杂度为 O(log(N)) 。
ZREVRANK key member
    返回成员在有序集合中的逆序排名，其中 排名按照元素的分值从大到小计算。
    排名以 0 开始。
    复杂度为 O(log(N)) 。
```
# 分值范围操作
```
ZRANGE key start stop [WITHSCORES]
    返回有序集合在按照分值从小到大排列元素（升序排列） 的情况下，索引 start 至索引 stop 范围之内的所有元素。
    两个索引都可以是正数或者 负数。当给定 WITHSCORES 选项时，命令会将元素和分值一并返回。
    命令的复杂度为 O(log(N)+M) ，N 为有序集合的基数，而 M 则为被返回元素的数量。
ZREVRANGE key start stop [WITHSCORES]
    返回有序集合在按照分值从大到小排列元素（降序排列） 的情况下，索引 start 至索引 stop 范围之内的所有元素。
    两个索引都可以是正数或者 负数。
    当给定 WITHSCORES 选项时，命令会把元素和分值一并返回。
    命令的复杂度为 O(log(N)+M) ，N 为有序集合的基数，而 M 则为被返回元素的数量。
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
    返回有序集合在按照分值升序排列元素的情况下，分值在 min 和 max 范围之内的所有元素。
    给定 WITHSCORES 选项时
    ，元素和分值会一并返回。给定 LIMIT 选项时，可以通过 offset 参数指定返
    回的结果集要跳过多少个元素，而 count参数则用于指定返回的元素数量。
    命令的复杂度为 O(log(N)+M)， N 为有序集合的基数，而 M 则为被返回元素的数量。
ZREVRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
    返回有序集合在按照分值降序排列元素的情况下，分值在 min 和 max 范围之内的所有元素。
    给定 WITHSCORES 选项时，元素和分值会一并返回。给定 LIMIT 选项，
    可以通过 offset 参数指定返回的结果集要跳过多少个元素，而 count 
    参数则用于指定返回的元素数量。
    命令的复杂度为 O(log(N)+M)， N 为有序集合的基数，而 M 则为被返回元素的数量。
ZCOUNT key min max
    返回有序集合在升序排列元素的情况下，分 值在 min 和 max 范围内的元素数量。
    命令的复杂度为 O(log(N))， N 为有序集合的基数。
ZREMRANGEBYRANK key start stop
    移除有序集合中，元素按升序 进行排列的情况下，指定排名范 围内的所有元素。
    命令的复杂度为 O(log(N)+M)， N 为有序集合的基数，而 M 则为被移除元素的数量。
ZREMRANGEBYSCORE key min max
    移除有序集合中，分值范围介于 min 和 max 之内的所有元素。
    命令的复杂度为 O(log(N)+M)，其中 N 为有序集合的基数，而 M 为被移除元素的数量。
```
# 集合的运算操作
```
ZUNIONSTORE destkey numkeys key [key ...]     
    计算并集 复杂度为O(N)+O(M log(M))， N 为参与并集计算的元素数量， M 为结果集的基数。
ZINTERSTORE destkey numkeys key [key ...] 
    计算交集 复杂度为O(N*K)+O(M*log(M))， N 为给定有序集合中，基数最小的有序集合的基数， 
    K 为给定有序集合的数量， M 为结果集的基数。
```

# 内部编码
>&nbsp;&nbsp;&nbsp;&nbsp;有序集合类型的内部编码有两种。

```
1.zipList(压缩列表):当有序集合类型元素个数小于hash-max-ziplist-entries配置(默认是512个),同时所有值都小于hash-max
-ziplist-value配置(默认是64个字节)时候,redis就会使用ziplist作为有序集合类型的内部实现,ziplist可以有效减少
内存的使用

2.skiplist(跳跃表):当有序集合类型无法满足ziplist的条件的时候,redis会使用skiplist作为有序集合的内部实现。这个时候ziplist
的读写效率会下降。
```