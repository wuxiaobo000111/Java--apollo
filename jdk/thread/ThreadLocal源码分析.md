[TOC]
### 源码详解

>这里针对的是set和get方法

### set方法
```java

ThreadLocal<Object> threadLocal = new ThreadLocal<>();
threadLocal.set("wuxiaobo");
threadLocal.set("wuxiaobo");
```

>首先看一下ThreadLocal的set方法

```java
  public void set(T value) {
        // 首先是拿到当前的线程
        Thread t = Thread.currentThread();
        // 然后通过getMap方法
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
  ThreadLocalMap getMap(Thread t) {
          return t.threadLocals;
    }  
    
  //ThreadLocalMap是ThreadLocal的一个静态内部类，其中看到
  // 存储结构是Entry[] table，而Entry的结构如下所示是一个虚引用，
  // key是一个ThreadLocal变量，是一个key-value结构的数据结构
  //    
  static class ThreadLocalMap {
          static class Entry extends WeakReference<ThreadLocal<?>> {
              /** The value associated with this ThreadLocal. */
              Object value;
  
              Entry(ThreadLocal<?> k, Object v) {
                  super(k);
                  value = v;
              }
          }
          
          private static final int INITIAL_CAPACITY = 16;
  
          /**
           * The table, resized as necessary.
           * table.length MUST always be a power of two.
           */
          private Entry[] table;
  
          /**
           * The number of entries in the table.
           */
          private int size = 0;
  
          /**
           * The next size value at which to resize.
           */
          private int threshold; // Default to 0
  }
```

>然后我们可以具体分析一下map.set(this, value);这个方法,可以看到每次set的时候，都会把ThreadLocal
这个对象本身传入进去，当做是Entry中的key
```java
private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            //通过key的哈希得到在数组中的索引值
            int i = key.threadLocalHashCode & (len-1);
            // 如果相等的话，表示原来有值，现在要做的是替换掉原来的值
            // 如果原来没有值，则就会进行赋值操作
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            //这个我也不知道是干嘛的......
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```
>通过之上的分析，我们可以得出结论，其实每次set的时候，就是存放在了当前线程的ThreadLocalMap中，
这个ThreadLocalMap其实是个数组对象，索引值是通过ThreadLocal本身通过hash得到的。这样的话，每次
创建一个ThreadLocal对象，只能是存在一个值。

### get方法
```java
public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```
>其实还是一样的，每次进来都会拿到当前线程，如果当前线程的ThreadLocalMap不是空，那么久
通过ThreadLocal对象获取数组下标，然后取值操作。否则调用setInitialValue(),其实源码中放入了一个
null值进去。
```java
 private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
    
 protected T initialValue() {
        return null;
    }
```


### 总结
>这就解释了为何ThreadLocal能创建线程独有的局部变量，可以在整个线程存活的过程中随时取用。
