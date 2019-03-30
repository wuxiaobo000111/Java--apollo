
# 事务

```java
  /**
     * 测试watch
     */
    public static void test1(Jedis jedis) throws InterruptedException {
        jedis.watch("watchkey");
        jedis.set("watchkey","false");
        Transaction transaction = jedis.multi();
        transaction.set("watchkey","123");
        transaction.set("company","嘀嗒出行");
        transaction.exec();
        jedis.unwatch();
        String wuxiaobo = jedis.get("company");
        System.out.println(wuxiaobo);
    }

    /**
     * redis的事务问题
     * @param jedis
     */
    public static void  test0(Jedis jedis) {
        //运行这个命令会清空redis数据库，请小心使用
        jedis.flushDB();
        //获取redis
        Transaction transaction = jedis.multi();
        transaction.set("wuxiaobo","123");
        transaction.set("company","吴晓波公司");
        transaction.exec();
        String wuxiaobo = jedis.get("wuxiaobo");
        System.out.println(wuxiaobo);
    }


    /**
     * 当有错误的时候，redis的事务是不执行的
     * @param jedis
     */
    public static void  test(Jedis jedis) {
        //运行这个命令会清空redis数据库，请小心使用
        jedis.flushDB();
        //获取redis
        Transaction transaction = jedis.multi();
        transaction.set("wuxiaobo","123");
        //当有报错的时候，事务是不执行的
        int i = 1 / 0;
        transaction.set("company","嘀嗒出行");
        transaction.exec();
        String wuxiaobo = jedis.get("wuxiaobo");
        System.out.println(wuxiaobo);
    }
```

# redis事务 原理

>MULTI、EXEC、DISCARD和WATCH命令是Redis事务功能的基础。Redis事务允许在一次单独的步骤中执行一组命令，并且可以保证如下两个重要事项：

>Redis会将一个事务中的所有命令序列化，然后按顺序执行。Redis不可能在一个Redis事务的执行过程中插入执行另一个客户端发出的请求。这样便能保证Redis将这些命令作为一个单独的隔离操作执行。

>在一个Redis事务中，Redis要么执行其中的所有命令，要么什么都不执行。因此，Redis事务能够保证原子性。

>case例子
```
 WATCH state
   value  = GET state;
   if value == 1
       UNWATCH state
       Return false;
   MULTI
   SET state 1
   result = EXEC
   if result == success
       return true;
   return false;
```

>MULTI：用于标记事务块的开始。Redis会将后续的命令逐个放入队列中，然后才能使用EXEC命令原子化地执行这个命令序列。
>EXEC：在一个事务中执行所有先前放入队列的命令，然后恢复正常的连接状态。
当使用WATCH命令时，只有当受监控的键没有被修改时，EXEC命令才会执行事务中的命令，这种方式利用了检查再设置（CAS）的机制。

>DISCARD：清除所有先前在一个事务中放入队列的命令，然后恢复正常的连接状态。如果使用了WATCH命令，那么DISCARD命令就会将当前连接监控的所有键取消监控。

>WATCH：当某个事务需要按条件执行时，就要使用这个命令将给定的键设置为受监控的。

> UNWATCH
清除所有先前为一个事务监控的键。如果你调用了EXEC或DISCARD命令，那么就不需要手动调用UNWATCH命令。

# Lua(尚未学习)
