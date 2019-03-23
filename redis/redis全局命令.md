
# 查看所有的键

    keys *

# 键总数
    dbsize

# 检查键是否存在
    exists key


# 删除键

del key1 key2 key3 [key......]

# 键过期

expire key seconds 

>&nbsp;&nbsp;&nbsp;&nbsp;ttl会返回键的剩余过期时间

```text
大于等于0的整数:键剩余的过期时间
-1: 没有设置过期时间
-2: 键不存在
```

# 键的数据结构类型

type key

