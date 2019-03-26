
# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;上次我们说了一下HashMap的实现,那么我们今天就来探究一下LinkedHashMap的实现吧。

# LinkedHashMap的数据结构

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/1.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;可以从上图中看到，LinkedHashMap数据结构相比较于HashMap来说，添加了双向指针，分别指向前一个节点——before和后一个节点——after，从而将所有的节点已链表的形式串联一起来，从名字上来看LinkedHashMap与HashMap有一定的联系，实际上也确实是这样，LinkedHashMap继承了HashMap，重写了HashMap的一部分方法，从而加入了链表的实现。让我们来看一下它们的继承关系。


```java
    public static void main(String[] args) {
        LinkedHashMap<Integer,Integer> map = new LinkedHashMap<>();

        for (int i= 10; i > 0; i--) {
            map.put(i,i);
        }

        System.out.println(map.toString());


        HashMap<Integer,Integer> hashMap = new HashMap<>();

        for (int i= 10; i > 0; i--) {
            hashMap.put(i,i);
        }

        System.out.println(hashMap.toString());

    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;从上面这个例子中会很容器看出LinkedHashMap和HashMap的区别。下面我们就深入源代码来分析一下LinkedHashMap的内部实现吧。

# 继承关系

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-26/2.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;我们可以看到LinkeHashMap继承了HashMap。

```java
    //对于数组中的节点新增了两个属性,一个是前置节点,一个是后置节点。最终的数据结构是如图一中所示。
    //
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

# LinkedHashMap新增属性
```java
    
    //头节点
    transient LinkedHashMap.Entry<K,V> head;

    
    //尾部节点
    transient LinkedHashMap.Entry<K,V> tail;

/**
 * 用来指定LinkedHashMap的迭代顺序，
 * true则表示按照基于访问的顺序来排列，意思就是最近使用的entry，放在链表的最末尾
 * false则表示按照插入顺序来
 */ 
    final boolean accessOrder;
```


>&nbsp;&nbsp;&nbsp;&nbsp;还有一些是关于LinkHashMap的内部类的,和HashMap中的类似，这里就不在讨论了,如果有兴趣,可以自己研究一下。



# LinkedHashMap的构造函数

```java 
    //无参构造,设置accessOrder是false
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }    
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
    //还是使用的是HashMap的putMapentries方法
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }     
```

>&nbsp;&nbsp;&nbsp;&nbsp关于accessOrder的使用可以参考如下代码。
```java
 LinkedHashMap<Integer,Integer> map = new LinkedHashMap<>(16,0.75f,true);
        for (int i= 10; i > 0; i--) {
            map.put(i,i);
        }
        System.out.println(map.toString());
        Integer integer = map.get(10);
        map.put(10,integer+1);
        System.out.println(map.toString());
```

# put方法

>&nbsp;&nbsp;&nbsp;&nbsp;底层代码中其实还是调用了HashMap中的putVal的方法。但是在LinkedHashMap中重写了了两个创建节点的方法。其他的逻辑保持不变
```java
    // 创建一个节点之后,会调用linkNodeLast()方法
     Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        //如果尾部节点是null,就是第一次添加的时候
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
    //存在hash冲突树化时候的节点 
    TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
        TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
        linkNodeLast(p);
        return p;
    }


```

# clear()方法

```java
    //hashMap中的clear方法
    public void clear() {
        super.clear();
        //头部节点和尾部节点都置空
        head = tail = null;
    }
```

# containsValue()方法

```java
    //从头部循环遍历,如果存在返回true,否则返回false
    public boolean containsValue(Object value) {
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
            V v = e.value;
            if (v == value || (value != null && value.equals(v)))
                return true;
        }
        return false;
    }


```

# get() 方法

```java
    //这里获取节点还是通过hash获取到索引,如果accessOrder
    //是true,就会把这个使用过的节点放在最后一个位置
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            //表示这个还没有添加元素,e是第一个,所以将头指针指向a
            if (b == null)
                head = a;
            //否则就将p的前一个节点,指向p的后一个节点    
            else
                b.after = a;
            //如果p的下一个节点不是null,则将a的前指针指向b    
            if (a != null)
                a.before = b;
            //如果是null,则说明这个map是个null的,将last指针指向b    
            else
                last = b;    
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```

# 总结

>&nbsp;&nbsp;&nbsp;&nbsp;从这里看出,其实LinkedHashMap其实是对HashMap的一种扩展,LinkedHashMap除去使用了数组+链表+红黑树的结构。同时把所有的节点都使用双向链表进行连接。作用可以提供更多的功能,比如可以把访问过的放在尾部的操作就是通过双向链表实现的。
