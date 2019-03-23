

# 散列（hash）

>&nbsp;&nbsp;&nbsp;&nbsp;一个散列由多个域值对（field-value pair）组成，散列的域和值都可以是文字、整数、浮点或者二进制数据。
>&nbsp;&nbsp;&nbsp;&nbsp;同一个散列里面的每个域必须是独一无二、各不相同 的，而域的值则没有这一要求，换句话说，不同域的值 可以是重复的。
>&nbsp;&nbsp;&nbsp;&nbsp;通过命令，用户可以对散列执行设置域值对、获取域的值、检查域是否存在等操作，也可以让 Redis 返回散列包含的所有域、所有值或者所有域值对。

# 基本操作
```
HSET key field value
    在散列键 key 中关联给定的域值对 field 和 value 。
    如果域 field 之前没有关联值，那么命令返回 1 ；
    如果域 field 已经有关联值，那么命令用新值覆盖旧值，并返回 0。
    复杂度为 O(1) 。
HGET key field
    返回散列键 key 中，域 field 所关联的值。如果域 field 没有关联值，那么返回 nil 。
    复杂度为 O(1) 。、
HSETNX key field value
    如果散列键 key 中，域 field 不存在（也即是， 还没有与之相关联的值），那么关联给定的域值对 field 和value 。
    如果域 field 已经有与之相关联的值，那么命令不做动作。
    复杂度为 O(1) 。
HEXISTS key field
    查看散列键 key 中，给定域 field 是否存在：存在返回 1 ，不存在返回 0 。
    复杂度为 O(1) 。
HDEL key field [field ...]
    删除散列键 key 中的一个或多个指定域，以及那些域的值。
    不存在的域将被忽略。命令返回被成功删除的域值对数量。
    复杂度为 O(N) ，N 为被删除的域值对数量。
HLEN key
    返回散列键 key 包含的域值对数量。
    复杂度为 O(1) 。
```    
# 批量操作
```
HMSET key field value [field value ...] 
    在散列键 key 中关联多个域值对，相当于同时执行多个 HSET 。复杂度为O(N) ，N 为输入的域值对数量。
HMGET key field [field ...] 
    返回散列键 key 中，一个或多个域的值，相当于同时执行多个 HGET 。复杂度O(N) ， N 为输入的域数量。
HKEYS key 
    返回散列键 key 包含的所有域。复杂度为O(N)，N 为被返回域的数量。
HVALS key 
    返回散列键 key 中，所有域的值。复杂度为O(N)，N 为被返回值的数量。
HGETALL key 
    返回散列键 key 包含的所有域值对。复杂度为O(N)，N 为被返回域值对的数量。
```
# 数字操作
```
HINCRBY key field increment 
    为散列键 key 中，域 field 的值加上整数增量 increment 。复杂度为O(1)
HINCRBYFLOAT key field increment 
    为散列键 key 中，域 field 的值加上浮点数增量 increment 。复杂度为O(1)
 ```   

>&nbsp;&nbsp;&nbsp;&nbsp;虽然 Redis 没有提供与以上两个命令相匹配的 HDECRBY 命令和 HDECRBYFLOAT 命令，但我们同样可以通过将 increment 设为负数来达到做减法的效果。hash类型和string类型的比较

# 使用hash的好处
>&nbsp;&nbsp;&nbsp;&nbsp;将数据放到同一个地方
散列可以让我们将一些相关的信息储存在同一个地方，而不是直接分散地储存在整个数据库里面，这不仅方便了数据管理，还可以尽量避免误操作发生。避免键名冲突减少内存的使用

# 结论

>&nbsp;&nbsp;&nbsp;&nbsp;只要有可能的话，就尽量使用散列键而不是字符串键来储存键值对数据，因为散列键管理方便、能够避免键名冲突、并且还能够节约内存。
一些没办法使用散列键来代替字符串键的情况：
1. 使用二进制位操作命令：因为 Redis 目前支持对字符串键进行 SETBIT、GETBIT、BITOP 等操作，
如果你想使用这些操作，那么只能使用字符串键。
2. 使用过期功能：Redis 的键过期功能目前只能对键进行过期操作，而不能对散列的域进行过期操
作，因此如果你要对键值对数据使用过期功能的话，那么只能把键值对储存在字符串里面。

# 内部编码

>&nbsp;&nbsp;&nbsp;&nbsp;Hash类型的内部编码有两种


```text
1.zipList(压缩列表):当Hash类型元素个数小于hash-max-ziplist-entries配置(默认是512个),同时所有值都小于hash-max
-ziplist-value配置(默认是64个字节)时候,redis就会使用ziplist作为哈希的内部实现,ziplist使用更加紧凑的结构实现
多个元素的连续存储,所以在节省内存方面比hashtable更加优秀。

2.hashtable(hash表):当hash类型无法满足ziplist的条件的时候,redis就会使用hashtable作为hash的内部实现,因为此时zip
list的读写效率会下降,而hashtable的读写时间复杂度为O(1)。
```

