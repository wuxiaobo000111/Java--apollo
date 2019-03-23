# 常用命令

## 基本操作

```text
SET key value
    将字符串键 key 的值设置为 value ，命令返回 OK 表示设置成功。
    如果字符串键 key 已经存在，那么用新值覆盖原来的旧值。
    复杂度为 O(1) 。
GET key
    返回字符串键 key 储存的值。
    复杂度为 O(1) 。
SETNX key value
    仅在键 key 不存在的情况下，将键 key 的值设置为 value ，效果和 SET key value NX 一样。
    NX 的意思为“Not eXists”（不存在）。
    键不存在并且设置成功时，命令返回 1；因为键已经存在而导致设置失败时，命令返回 0 。
MSET key value [key value ...] 
     一次为一个或多个字符串键设置值，效果和同时执行多个SET 命令一样。命令返回 OK 。
    复杂度为O(N)，N 为要设置的字符串键数量。
MGET key [key ...] 
    一次返回一个或多个字符串键的值，效果和同时执行多GET 命令一样。
    复杂度O(N)，N 为要获取的字符串键数量。
MSETNX key value [key value ...]
    只有在所有给定键都不存在的情况下， MSETNX 会为所有给定键设置值，效果和同时执行多个
    SETNX 一样。如果给定的键至少有一个是存在的，那么 MSETNX 将不执行任何设置操作。
    返回 1 表示设置成功，返回 0 表示设置失败。复杂度为 O(N) ， N 为给定的键数量。
GETSET key new-value
    将字符串键的值设置为 new-value ，并返回字符串键在设置新值之前储存的旧值（old value）。
    复杂度为 O(1) 。
APPEND key value
    将值 value 推入到字符串键 key 已储存内容的末尾。
    O(N)， 其中 N 为被推入值的长度。
STRLEN key
    返回字符串键 key 储存的值的长度。
    因为 Redis 会记录每个字符串值的长度，所以获取该值的复杂度为 O(1) 。

```    
## 索引和范围
```
字符串的索引（ index）以 0 为开始，从字符串的开头向字符串的结尾依次递增，字符串第一个字符的索
引为 0 ，字符串最后一个字符的索引为 N-1 ，其中 N 为字符串的长度。
除了（正数）索引之外，字符串还有负数索引：负数索引以 -1 为开始，从字符串的结尾向字符串的开头
依次递减，字符串的最后一个字符的索引为 -N ，其中 N 为字符串的长度。
SETRANGE key index value
    从索引 index 开始，用 value 覆写（overwrite）给定键 key 所储存的字符串值。只接受正数索引。
    命令返回覆写之后，字符串值的长度。复杂度为 O(N)， N 为 value 的长度。  
GETRANGE key start end
    返回键 key 储存的字符串值中，位于 start 和 end 两个索引之间的内容（闭区间，start 和 end 会被包括
    在内）。和 SETRANGE 只接受正数索引不同， GETRANGE 的索引可以是正数或者负数。
    复杂度为 O(N) ， N 为被选中内容的长度。
```


## 数字操作
```
只要储存在字符串键里面的值可以被解释为 64 位整数，或者 IEEE-754 标准的 64 位浮点数，
那么用户就可以对这个字符串键执行针对数字值的命令。
INCRBY key increment
    将 key 所储存的值加上增量 increment ，命令返回操作执行之后，键 key 的当前值。复杂度为O(1)
DECRBY key decrement
    将 key 所储存的值减去减量 decrement ，命令返回操作执行之后，键 key的当前值。复杂度为O(1)
INCR key 
    等同于执行 INCRBY key 1 复杂度为O(1)
DECR key 
    等同于执行 DECRBY key 1 复杂度为O(1)
INCRBYFLOAT key increment
    为字符串键 key 储存的值加上浮点数增量 increment ，命令返回操作执行之后，键 key 的值。
    没有相应的 DECRBYFLOAT ，但可以通过给定负值来达到 DECRBYFLOAT 的效果。
复杂度为 O(1) 。
注意事项

即使字符串键储存的是数字值，它也可以执行 APPEND、STRLEN、SETRANGE 和 GETRANGE 。
当用户针对一个数字值执行这些命令的时候，Redis 会先将数字值转换为字符串，然后再执行命令。
```

## 二进制数据操作
```
SET 、GET 、SETNX、APPEND等命令同样可以用于设置二进制数据。
和储存文字时一样，字符串键在储存二进制位时，索引也是从 0 开始的。
但是和储存文字时，索引从左到右依次递增不同，当字符串键储存的是二进制位时，二进制位的索引会
从左到右依次递减。
SETBIT key index value
    将给定索引上的二进制位的值设置为 value ，命令返回被设置的位原来储存的旧值。
    复杂度为 O(1) 。
GETBIT key index
    返回给定索引上的二进制位的值。
    复杂度为 O(1) 。
BITCOUNT key [start] [end]
    计算并返回字符串键储存的值中，被设置为 1 的二进制位的数量。
    一般情况下，给定的整个字符串键都会进行计数操作，    但通过指定额外的 start 或 end 参数，可以让计
    数只在特定索引范围的位上进行。
    start 和 end 参数的设置和 GETRANGE 命令类似，都可以使用负数值：比如 -1 表示最后一个位，而 -2
    表示倒数第二个位，以此类推。
BITOP operation destkey key [key ...]
    对一个或多个保存二进制位的字符串键执行位元操作，并将结果保存到 destkey 上。
    operation 可以是 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种：
    命令
```

# 内部编码

>&nbsp;&nbsp;&nbsp;&nbsp;string类型的内部编码有三种

```
int : 8个字节的长整型
embstr : 小于等于39个字节的字符串
raw : 大于39个字节的字符串
```