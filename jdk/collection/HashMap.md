# HashMap概述

>&nbsp;&nbsp;&nbsp;&nbsp;HashMap是我们在工作中比较常用的一个类,但是估计很少有人知道HashMap中的原理。这篇文章就来讲述一下HashMap的源码分析。首先来看一个easy的例子。
```java
 public static void main(String[] args) {

        HashMap<Integer,Object> map = new HashMap<>(16);

        for (int i =0; i<100; i++) {
            map.put(i,i);
        }
        System.out.println(map.toString());
    }
```
>&nbsp;&nbsp;&nbsp;&nbsp;怎么样是不是很简单，首先先来看一下HashMap的继承关系和属性。然后通过源码分析一下，上边几行简单的代码都做了什么。
# HashMap的继承关系

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-19/1.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;Cloneable和Serializable这两个接口就不说的。主要是实现了Map这个接口。


# HashMap的属性
```java
    /** HashMap指定的默认大小，源码备注上说必须是2的次方，之后会解释一下为什么是2的次方
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16



    /**默认的扩容大小，就是底层数组实际使用的长度超过
    * 数组长度的0.75的时候,就会进行扩容操作。
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;


     /**
     * 将Map转化为Set结合,如何转化会在下面进行讲解
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-19/2.jpg?raw=true)


```java
    /** 底层数组的大小
     * The number of key-value mappings contained in this map.
     */
    transient int size;

    //底层数组被修改的次数 
    transient int modCount; 
    //扩容边界
    final float loadFactor;

    /**
     *最大的容量
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 当我第一眼看到这里的时候其实我也是一脸蒙蔽，不知道是干嘛的，这里先放过去
    static final int MIN_TREEIFY_CAPACITY = 64;
    //真正保存的数组，是一个Node类型的数组，
    transient Node<K,V>[] table;    

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;
```

>&nbsp;&nbsp;&nbsp;&nbsp;翻译一下上面这段话的意思:容器(桶,也就是我们在JDK1.7的时候当hash碰撞的时候,会使用一个链表来存储)大小的阈值使用的是树结构而不是使用一个List结构，当添加一个元素到容器中，并且这个桶的大小至少是8的时候，这个桶将会被转化为一棵树。(后面的我也看不懂了......)

```java

    static final int UNTREEIFY_THRESHOLD = 6;
```

>&nbsp;&nbsp;&nbsp;&nbsp;当扩容时，桶中元素个数小于这个值就会把树形的桶元素 还原（切分）为链表结构。

>&nbsp;&nbsp;&nbsp;&nbsp;到目前为止是一脸蒙蔽。看源码就是硬着头皮往下看。


# 数组中的元素Node

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

>这个Node实现了Map.Entry接口。这个数据结构长得其实像是个单链表的数组结构。通过这个Node节点属性,其实我们大致可以猜出HashMap的数组结构.

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-19/4.jpg?raw=true)

>灵魂画图,实际的结果等待我们自己去验证。


# KeySet
```java
    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super K> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.key);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;顾名思义就是key的set集合，其中有调用HashMap自身的方法，其实想想也很easy,比如判断key是否存在，那么就判断Node是否存在。当移除一个Key的时候其实也是移除一个Node节点。使用方法如下所示。

```java
        HashMap<Integer,Object> map = new HashMap<>(16);

        for (int i =0; i<10; i++) {
            map.put(i,i);
        }

        Set<Integer> set = map.keySet();
        System.out.println(set.size());

        System.out.println(set.contains(1));

        set.remove(1);
        System.out.println(map.toString());

        System.out.println("一道华丽的分割线==============================");
        set.forEach(integer -> {
            System.out.print(integer);
        });
        System.out.println();
```


# Values

```java
    public Collection<V> values() {
        Collection<V> vs = values;
        if (vs == null) {
            vs = new Values();
            values = vs;
        }
        return vs;
    }

    final class Values extends AbstractCollection<V> {
        //返回的是HasHMap的实际大小
        public final int size()                 { return size; }
        //调用HashMap的clear方法
        public final void clear()               { HashMap.this.clear(); }
        //迭代器获得是一个ValueIterator对象
        public final Iterator<V> iterator()     { return new ValueIterator(); }
        public final boolean contains(Object o) { return containsValue(o); }
        public final Spliterator<V> spliterator() {
            return new ValueSpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super V> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.value);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }

    //clear就是将数组置为空    
     public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
    // ValueIterator继承HashIterator,同时实现了Iterator
    final class ValueIterator extends HashIterator
        implements Iterator<V> {
        public final V next() { return nextNode().value; }
    }

    abstract class HashIterator {
        //下一个节点
        Node<K,V> next;        // next entry to return
        //当前的节点
        Node<K,V> current;     // current entry
        // 期望的修改次数，用于多线程中，抛出异常使用
        int expectedModCount;  // for fast-fail
        //当前的节点索引,初始化的时候为索引值加1
        int index;             // current slot

        HashIterator() {
            expectedModCount = modCount;
            Node<K,V>[] t = table;
            current = next = null;
            index = 0;
            if (t != null && size > 0) { // advance to first entry
                do {} while (index < t.length && (next = t[index++]) == null);
            }
        }

        public final boolean hasNext() {
            return next != null;
        }

        //获取下一个节点, 然后index属性加1
        final Node<K,V> nextNode() {
            Node<K,V>[] t;
            Node<K,V> e = next;
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            if ((next = (current = e).next) == null && (t = table) != null) {
                do {} while (index < t.length && (next = t[index++]) == null);
            }
            return e;
        }

        //删除一个元素，这个地方其实HashMap底层删除的原理，之后的地方会讲到
        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            expectedModCount = modCount;
        }
    }

```

>&nbsp;&nbsp;&nbsp;&nbsp;具体使用方法如下所示
```java
        HashMap<Integer,Object> map = new HashMap<>(16);

        for (int i =0; i<10; i++) {
            map.put(i,i);
        }

        Collection<Object> values = map.values();
        Iterator<Object> iterator = values.iterator();
        while (iterator.hasNext()) {
            Integer next = (Integer) iterator.next();
            System.out.println(next);
        }

        System.out.println(values.size());
```

>&nbsp;&nbsp;&nbsp;&nbsp;这个内部类就是获取Value的值。遍历,其实也是类似于HashMap本身的操作。在源码中可以看到调用的很多的都是HashMap本身的方法。作为一个内部类，其实也是可以调用HashMap本身的方法。


# EntrySet
```java
    final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<Map.Entry<K,V>> iterator() {
            return new EntryIterator();
        }
        public final boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Node<K,V> candidate = getNode(hash(key), key);
            return candidate != null && candidate.equals(e);
        }
        public final boolean remove(Object o) {
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>) o;
                Object key = e.getKey();
                Object value = e.getValue();
                return removeNode(hash(key), key, value, true, true) != null;
            }
            return false;
        }
        public final Spliterator<Map.Entry<K,V>> spliterator() {
            return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;和KetSet以及Values的用法差不多。这里不多描述了。



#

```java
    // 这个是LinkedHashMap中的Entry,又一次继承了HashMap的Node。
    //通过上面源码分析，Node的属性有hash,value,nextNode,key这四
    //个属性值。LinkedHashMap中的Entry有加入了两个节点,Before和
    //和ater,这样就形成了一个双向链表结构
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```
![](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-21/1.png?raw=true "")

>&nbsp;&nbsp;&nbsp;&nbsp;如上图所示的一个结构。


```java


    // TreeNode节点继承了LinkedHashMap.Entry，形成了一个二叉树的结构。
    //(其实真正的结构是红黑树,这里又要去复习一下红黑树的东东了。水平有限,暂时先跳过......)
    // 
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        /**
         * Returns root of tree containing this node.
         */
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }

        /**
         * Ensures that the given root is the first node of its bin.
         */
        static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
            int n;
            if (root != null && tab != null && (n = tab.length) > 0) {
                int index = (n - 1) & root.hash;
                TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
                if (root != first) {
                    Node<K,V> rn;
                    tab[index] = root;
                    TreeNode<K,V> rp = root.prev;
                    if ((rn = root.next) != null)
                        ((TreeNode<K,V>)rn).prev = rp;
                    if (rp != null)
                        rp.next = rn;
                    if (first != null)
                        first.prev = root;
                    root.next = first;
                    root.prev = null;
                }
                assert checkInvariants(root);
            }
        }

        /**
         * Finds the node starting at root p with the given hash and key.
         * The kc argument caches comparableClassFor(key) upon first use
         * comparing keys.
         */
        final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
            TreeNode<K,V> p = this;
            do {
                int ph, dir; K pk;
                TreeNode<K,V> pl = p.left, pr = p.right, q;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.find(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
            return null;
        }

        /**
         * Calls find for root node.
         */
        final TreeNode<K,V> getTreeNode(int h, Object k) {
            return ((parent != null) ? root() : this).find(h, k, null);
        }

        /**
         * Tie-breaking utility for ordering insertions when equal
         * hashCodes and non-comparable. We don't require a total
         * order, just a consistent insertion rule to maintain
         * equivalence across rebalancings. Tie-breaking further than
         * necessary simplifies testing a bit.
         */
        static int tieBreakOrder(Object a, Object b) {
            int d;
            if (a == null || b == null ||
                (d = a.getClass().getName().
                 compareTo(b.getClass().getName())) == 0)
                d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                     -1 : 1);
            return d;
        }

        /**
         * Forms tree of the nodes linked from this node.
         * @return root of tree
         */
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (root == null) {
                    x.parent = null;
                    x.red = false;
                    root = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);

                        TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            moveRootToFront(tab, root);
        }

        /**
         * Returns a list of non-TreeNodes replacing those linked from
         * this node.
         */
        final Node<K,V> untreeify(HashMap<K,V> map) {
            Node<K,V> hd = null, tl = null;
            for (Node<K,V> q = this; q != null; q = q.next) {
                Node<K,V> p = map.replacementNode(q, null);
                if (tl == null)
                    hd = p;
                else
                    tl.next = p;
                tl = p;
            }
            return hd;
        }

        /**
         * Tree version of putVal.
         */
        final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            TreeNode<K,V> root = (parent != null) ? root() : this;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            return q;
                    }
                    dir = tieBreakOrder(k, pk);
                }

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        ((TreeNode<K,V>)xpn).prev = x;
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }

        /**
         * Removes the given node, that must be present before this call.
         * This is messier than typical red-black deletion code because we
         * cannot swap the contents of an interior node with a leaf
         * successor that is pinned by "next" pointers that are accessible
         * independently during traversal. So instead we swap the tree
         * linkages. If the current tree appears to have too few nodes,
         * the bin is converted back to a plain bin. (The test triggers
         * somewhere between 2 and 6 nodes, depending on tree structure).
         */
        final void removeTreeNode(HashMap<K,V> map, Node<K,V>[] tab,
                                  boolean movable) {
            int n;
            if (tab == null || (n = tab.length) == 0)
                return;
            int index = (n - 1) & hash;
            TreeNode<K,V> first = (TreeNode<K,V>)tab[index], root = first, rl;
            TreeNode<K,V> succ = (TreeNode<K,V>)next, pred = prev;
            if (pred == null)
                tab[index] = first = succ;
            else
                pred.next = succ;
            if (succ != null)
                succ.prev = pred;
            if (first == null)
                return;
            if (root.parent != null)
                root = root.root();
            if (root == null || root.right == null ||
                (rl = root.left) == null || rl.left == null) {
                tab[index] = first.untreeify(map);  // too small
                return;
            }
            TreeNode<K,V> p = this, pl = left, pr = right, replacement;
            if (pl != null && pr != null) {
                TreeNode<K,V> s = pr, sl;
                while ((sl = s.left) != null) // find successor
                    s = sl;
                boolean c = s.red; s.red = p.red; p.red = c; // swap colors
                TreeNode<K,V> sr = s.right;
                TreeNode<K,V> pp = p.parent;
                if (s == pr) { // p was s's direct parent
                    p.parent = s;
                    s.right = p;
                }
                else {
                    TreeNode<K,V> sp = s.parent;
                    if ((p.parent = sp) != null) {
                        if (s == sp.left)
                            sp.left = p;
                        else
                            sp.right = p;
                    }
                    if ((s.right = pr) != null)
                        pr.parent = s;
                }
                p.left = null;
                if ((p.right = sr) != null)
                    sr.parent = p;
                if ((s.left = pl) != null)
                    pl.parent = s;
                if ((s.parent = pp) == null)
                    root = s;
                else if (p == pp.left)
                    pp.left = s;
                else
                    pp.right = s;
                if (sr != null)
                    replacement = sr;
                else
                    replacement = p;
            }
            else if (pl != null)
                replacement = pl;
            else if (pr != null)
                replacement = pr;
            else
                replacement = p;
            if (replacement != p) {
                TreeNode<K,V> pp = replacement.parent = p.parent;
                if (pp == null)
                    root = replacement;
                else if (p == pp.left)
                    pp.left = replacement;
                else
                    pp.right = replacement;
                p.left = p.right = p.parent = null;
            }

            TreeNode<K,V> r = p.red ? root : balanceDeletion(root, replacement);

            if (replacement == p) {  // detach
                TreeNode<K,V> pp = p.parent;
                p.parent = null;
                if (pp != null) {
                    if (p == pp.left)
                        pp.left = null;
                    else if (p == pp.right)
                        pp.right = null;
                }
            }
            if (movable)
                moveRootToFront(tab, r);
        }

        /**
         * Splits nodes in a tree bin into lower and upper tree bins,
         * or untreeifies if now too small. Called only from resize;
         * see above discussion about split bits and indices.
         *
         * @param map the map
         * @param tab the table for recording bin heads
         * @param index the index of the table being split
         * @param bit the bit of hash to split on
         */
        final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
            TreeNode<K,V> b = this;
            // Relink into lo and hi lists, preserving order
            TreeNode<K,V> loHead = null, loTail = null;
            TreeNode<K,V> hiHead = null, hiTail = null;
            int lc = 0, hc = 0;
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                next = (TreeNode<K,V>)e.next;
                e.next = null;
                if ((e.hash & bit) == 0) {
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    ++lc;
                }
                else {
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    ++hc;
                }
            }

            if (loHead != null) {
                if (lc <= UNTREEIFY_THRESHOLD)
                    tab[index] = loHead.untreeify(map);
                else {
                    tab[index] = loHead;
                    if (hiHead != null) // (else is already treeified)
                        loHead.treeify(tab);
                }
            }
            if (hiHead != null) {
                if (hc <= UNTREEIFY_THRESHOLD)
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    tab[index + bit] = hiHead;
                    if (loHead != null)
                        hiHead.treeify(tab);
                }
            }
        }

        /* ------------------------------------------------------------ */
        // Red-black tree methods, all adapted from CLR

        static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root,
                                              TreeNode<K,V> p) {
            TreeNode<K,V> r, pp, rl;
            if (p != null && (r = p.right) != null) {
                if ((rl = p.right = r.left) != null)
                    rl.parent = p;
                if ((pp = r.parent = p.parent) == null)
                    (root = r).red = false;
                else if (pp.left == p)
                    pp.left = r;
                else
                    pp.right = r;
                r.left = p;
                p.parent = r;
            }
            return root;
        }

        static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root,
                                               TreeNode<K,V> p) {
            TreeNode<K,V> l, pp, lr;
            if (p != null && (l = p.left) != null) {
                if ((lr = p.left = l.right) != null)
                    lr.parent = p;
                if ((pp = l.parent = p.parent) == null)
                    (root = l).red = false;
                else if (pp.right == p)
                    pp.right = l;
                else
                    pp.left = l;
                l.right = p;
                p.parent = l;
            }
            return root;
        }

        static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                                    TreeNode<K,V> x) {
            x.red = true;
            for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
                if ((xp = x.parent) == null) {
                    x.red = false;
                    return x;
                }
                else if (!xp.red || (xpp = xp.parent) == null)
                    return root;
                if (xp == (xppl = xpp.left)) {
                    if ((xppr = xpp.right) != null && xppr.red) {
                        xppr.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        if (x == xp.right) {
                            root = rotateLeft(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {
                            xp.red = false;
                            if (xpp != null) {
                                xpp.red = true;
                                root = rotateRight(root, xpp);
                            }
                        }
                    }
                }
                else {
                    if (xppl != null && xppl.red) {
                        xppl.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        if (x == xp.left) {
                            root = rotateRight(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {
                            xp.red = false;
                            if (xpp != null) {
                                xpp.red = true;
                                root = rotateLeft(root, xpp);
                            }
                        }
                    }
                }
            }
        }

        static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root,
                                                   TreeNode<K,V> x) {
            for (TreeNode<K,V> xp, xpl, xpr;;)  {
                if (x == null || x == root)
                    return root;
                else if ((xp = x.parent) == null) {
                    x.red = false;
                    return x;
                }
                else if (x.red) {
                    x.red = false;
                    return root;
                }
                else if ((xpl = xp.left) == x) {
                    if ((xpr = xp.right) != null && xpr.red) {
                        xpr.red = false;
                        xp.red = true;
                        root = rotateLeft(root, xp);
                        xpr = (xp = x.parent) == null ? null : xp.right;
                    }
                    if (xpr == null)
                        x = xp;
                    else {
                        TreeNode<K,V> sl = xpr.left, sr = xpr.right;
                        if ((sr == null || !sr.red) &&
                            (sl == null || !sl.red)) {
                            xpr.red = true;
                            x = xp;
                        }
                        else {
                            if (sr == null || !sr.red) {
                                if (sl != null)
                                    sl.red = false;
                                xpr.red = true;
                                root = rotateRight(root, xpr);
                                xpr = (xp = x.parent) == null ?
                                    null : xp.right;
                            }
                            if (xpr != null) {
                                xpr.red = (xp == null) ? false : xp.red;
                                if ((sr = xpr.right) != null)
                                    sr.red = false;
                            }
                            if (xp != null) {
                                xp.red = false;
                                root = rotateLeft(root, xp);
                            }
                            x = root;
                        }
                    }
                }
                else { // symmetric
                    if (xpl != null && xpl.red) {
                        xpl.red = false;
                        xp.red = true;
                        root = rotateRight(root, xp);
                        xpl = (xp = x.parent) == null ? null : xp.left;
                    }
                    if (xpl == null)
                        x = xp;
                    else {
                        TreeNode<K,V> sl = xpl.left, sr = xpl.right;
                        if ((sl == null || !sl.red) &&
                            (sr == null || !sr.red)) {
                            xpl.red = true;
                            x = xp;
                        }
                        else {
                            if (sl == null || !sl.red) {
                                if (sr != null)
                                    sr.red = false;
                                xpl.red = true;
                                root = rotateLeft(root, xpl);
                                xpl = (xp = x.parent) == null ?
                                    null : xp.left;
                            }
                            if (xpl != null) {
                                xpl.red = (xp == null) ? false : xp.red;
                                if ((sl = xpl.left) != null)
                                    sl.red = false;
                            }
                            if (xp != null) {
                                xp.red = false;
                                root = rotateRight(root, xp);
                            }
                            x = root;
                        }
                    }
                }
            }
        }

        /**
         * Recursive invariant check
         */
        static <K,V> boolean checkInvariants(TreeNode<K,V> t) {
            TreeNode<K,V> tp = t.parent, tl = t.left, tr = t.right,
                tb = t.prev, tn = (TreeNode<K,V>)t.next;
            if (tb != null && tb.next != t)
                return false;
            if (tn != null && tn.prev != t)
                return false;
            if (tp != null && t != tp.left && t != tp.right)
                return false;
            if (tl != null && (tl.parent != t || tl.hash > t.hash))
                return false;
            if (tr != null && (tr.parent != t || tr.hash < t.hash))
                return false;
            if (t.red && tl != null && tl.red && tr != null && tr.red)
                return false;
            if (tl != null && !checkInvariants(tl))
                return false;
            if (tr != null && !checkInvariants(tr))
                return false;
            return true;
        }
    }
```


# HashMap的构造函数

```java
    //最终调用的都是这个构造方法,initialCapacity指定的是
    //数组的大小,loadFactor装载因子或者是负载因子
    // 如果不指定initialCapacity，则默认是大小是16,
    // 如果不指定loadFactor,则默认大小是0.75
    // 其实可以看到在初始化的时候并没有真正意义上的初始化。
    public HashMap(int initialCapacity, float loadFactor) {
        //首先校验参数
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
    
    //获取初始化容量最接近的2次幂的值
    //但是在看源码的时候,发现了一个非常奇怪的问题,
    //比如指定大小是13,但是在Debug的时候发现这个值是11,
    //而且更为奇怪的是,无论我指定为什么值,这个值都会是11
    //如果有人知道答案的话,可以提一个issue。
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    //如果放入的是一个集合,就会去执行putMapentries方法
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }

   
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        // 获取集合的大小
        int s = m.size();
        if (s > 0) {
            // 如果底层数组是null,则
            if (table == null) { // pre-size
                //计算集合大小除以负载因子,
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                // 如果t的值大于现在的阈值,那么就会进行一次扩展,获取新的阈值         
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            //如果集合大小大于阈值,则会执行resize方法
            else if (s > threshold)
                resize();
            //遍历集合,执行putVal()方法    
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }

    //这里我们看一下resize()这个方法
    final Node<K,V>[] resize() {
        //将oldTab指向table
        Node<K,V>[] oldTab = table;
        // oldCap是旧的容量大小
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //oldThr是旧的阈值大小
        int oldThr = threshold;
        // 初始化新的容量大小和阈值大小
        int newCap, newThr = 0;
        // 如果原来容量大于0
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 如果旧的容量左右1位(其实就是乘以2)小于最大容量
            //并且旧的大小大于等于默认大小
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                //则新的阈值就是原来阈值的两倍     
                newThr = oldThr << 1; // double threshold
        }
        //如果旧容量为0并且旧阈值大于0，那么新的容量指定为旧的阈值
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        //否则就指定默认的    
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //如果阈值还没有初始化,则就初始化阈值。
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        //遍历旧的数组
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    //先将旧的置为空
                    oldTab[j] = null;
                    //表示是没有hash冲突
                    if (e.next == null)
                        //就通过运算找到新的索引值,插入到新的数组中去
                        newTab[e.hash & (newCap - 1)] = e;
                    //否则如果e是TreeNode,表示这个桶中已经红黑树化,
                    //    
                    else if (e instanceof TreeNode)
                        //扩容的时候转移节点(根据rehash的特殊性,把红黑树拆成两个链表,然后再根据两个链表的长度决定是否树化或者链表化.)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        //有Hash冲突,桶中的结构是链表的形式
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            //获取当前节点的下一个节点
                            next = e.next;
                            ////e.hash & oldCap 说明e.hash后的值小于oldCap,即使扩充了,也是扩充
                            //链表的后半部分,前面已经有的元素还是在原来的位置
                            if ((e.hash & oldCap) == 0) {
                                ////如果loTail==null说明是第一个元素,把这个元素赋值给loHead
                                if (loTail == null)
                                    loHead = e;
                                else
                                    //如果不是第一个元素,就把他加到后面
                                    loTail.next = e;
                                loTail = e;
                            }
                            ////如果走下面的代码,则说明,新的元素的key值的hash已经超过了oldCap,所有要加入到新库充的链表中
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            // //下标j + oldCap指向第二个链表的头,原因是e.hash & oldCap ==0 说明,之前的容量还没
                            //有填满 知道 !=0 的时候,才表示之前的元素已经填满,所以下标, 就编程了j+oldCap
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

 

    //如下这个方法是真正的将K-V放入到底层数组中
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果table为null,表示是第一次添加。则使用resize其实是用来进行初始化。
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //根据key的值计算出来底层数组上的索引位置是null,则就添加进去,这个时候是没有hash碰撞的。    
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            //如果发生了Hash碰撞,则会走下面的逻辑
            Node<K,V> e; K k;
            //碰撞时候,如果key的值等于,则会直接替换
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果是TreeNode，表示这个桶的位置已经被树化,则加入红黑树中    
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //这个时候桶中的结构还是链表的结构    
            else {
                for (int binCount = 0; ; ++binCount) {
                    //如果在索引上的节点没有next的时候
                    if ((e = p.next) == null) {
                        //会将next设置一下
                        p.next = newNode(hash, key, value, null);
                        //如果binCount的值大于TREEIFY_THRESHOLD(这个值是8,这个时候会将链表进行树化)
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果e和传入的hash,key,value值相等，什么都不做
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //存在被匹配的
            if (e != null) { // existing mapping for key
                //返回就只
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //如果当前大小大于阈值了,这个时候会进行扩容的操作
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        //这个时候表示没有旧值，则直接返回一个null
        return null;
    }


```

# hash方法

```java
   //HashMap的Hash函数,可以看到是通过Key的hash值异或Key的hash值有移动16位
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;上边这段代码叫扰动函数。大家都知道上面代码里的key.hashCode()函数调用的是key键值类型自带的哈希函数，返回int型散列值。


>&nbsp;&nbsp;&nbsp;&nbsp;理论上散列值是一个int型，如果直接拿散列值作为下标访问HashMap主数组的话，考虑到2进制32位带符号的int表值范围从-2147483648到2147483648。前后加起来大概40亿的映射空间。只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。



>&nbsp;&nbsp;&nbsp;&nbsp;但问题是一个40亿长度的数组，内存是放不下的。你想，HashMap扩容之前的数组初始大小才16。所以这个散列值是不能直接拿来用的。如下图所示:下图中解释了HashMap中如何通过key来锁定到数组的下标。就是为了混合原始哈希码的高位和低位,以此来加大低位的随机性。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-22/1.jpg?raw=true)


# getNode方法

>&nbsp;&nbsp;&nbsp;&nbsp;在HashMap中使用了很多关于获取节点数据的地方,所以这里说一下getNode的方法。传入的其中一个参数就是扰动函数获取的hash值,另外一个就是key。

```java
 final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //将数组的引用给tab,如果数组不为空并且这个桶存在
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //如果桶中的第一个元素就是要获取的值    
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //如果有Hash冲突的时候    
            if ((e = first.next) != null) {
                //如果是红黑树结构,则通过getTreeNode方法获取 
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //否则就是链表，通过遍历链表的方法一层一层的往下走,如果存在这个key,就会返回值    
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        //否则就表示这个key所对应的key-value不存在,直接返回一个null
        return null;
    }
```

# remove方法

```java
    // remove方法返回值是要删除的key-value中的value
    public V remove(Object key) {
        Node<K,V> e;
        //在源码中调用的是removeNode方法
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }

    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        //如果在数组中存在 
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            //这个判断表示要删除的是否在数组中,如果是在数组中,则把将要删除的节点赋值给node
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            //如果执行到这里,则证明存在Hash碰撞,    
            else if ((e = p.next) != null) {
                //如果节点是红黑树,则通过getTreeNode方法拿到这个节点
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                //否则通过循环遍历的方式找到这个节点    
                else {
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            //如果节点存在
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                //如果是红黑树,则通过removeTreeNode方法进行删除                    
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                //如果是在数组中发现的,则将这个数组节点上的数组设置为他的next    
                else if (node == p)
                    tab[index] = node.next;
                //执行到下面这个方法,z则证明是在链表结构中发现的,只需要将前一个节点的next指向这个节点的next就行了    
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }

```

# 探究(当发生Hash冲突的时候,什么时候是链表结构,什么时候是红黑树结构)


```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;putVa的方法我们已经分析过了,在putVal方法中有这么一段代码如下所示:
```java
 if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //如果代码走到这个else中表示现在还是个链表结构    
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
```
>&nbsp;&nbsp;&nbsp;&nbsp;  if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st这么一行代码，之后就会调用treeifyBin的方法,也就是这个桶中的元素的大小大于等于8的时候会调用treeifyBin()方法,在这个方法中,当数组的长度小于MIN_TREEIFY_CAPACITY这个数的时候,会调用resize()方法。resize()方法我们也进行了分析。MIN_TREEIFY_CAPACITY这个值是64,当大于64的时候，才会循环调用replacementTreeNode()方法。从这里我们可以得出,只有当Hash冲突链表的大小大于等于8的时候,同时数组的大小大于等于64的时候,才会进行树化。所以这里你看懂了吗......


# 补充

>&nbsp;&nbsp;&nbsp;&nbsp;我们都知道HashMap在多线程中是不安全的。测试代码如下

```java
package com.bobo.basic.collection;

import java.util.HashMap;
import java.util.Random;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class HasHMapTest {

    public static void main(String[] args) {

        HashMap<Integer, Object> map = new HashMap<>(2);

        AtomicInteger integer = new AtomicInteger();
        /*HashMapThread hashMapThread1 = new HashMapThread(map,integer);
        HashMapThread hashMapThread2 = new HashMapThread(map,integer);
        HashMapThread hashMapThread3 = new HashMapThread(map,integer);
        HashMapThread hashMapThread4 = new HashMapThread(map,integer);
        HashMapThread hashMapThread5 = new HashMapThread(map,integer);
        HashMapThread hashMapThread6 = new HashMapThread(map,integer);
        HashMapThread hashMapThread7 = new HashMapThread(map,integer);
        HashMapThread hashMapThread8 = new HashMapThread(map,integer);
        HashMapThread hashMapThread9 = new HashMapThread(map,integer);


        hashMapThread1.start();
        hashMapThread2.start();
        hashMapThread3.start();
        hashMapThread4.start();
        hashMapThread5.start();
        hashMapThread6.start();
        hashMapThread7.start();
        hashMapThread8.start();
        hashMapThread9.start();*/

        ExecutorService executorService = new ThreadPoolExecutor(10,10,10L,TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(100));
        for (int i=0; i<10; i++) {
            executorService.execute(new HashMapThread(map,integer));
        }

        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(map.size());

        System.out.println(map);

        executorService.shutdown();
    }
}


class HashMapThread implements Runnable {
    private HashMap<Integer, Object> hashMap;

    public HashMapThread(HashMap<Integer, Object> hashMap,AtomicInteger atomicInteger) {
        this.hashMap = hashMap;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            int i1 = new Random().nextInt(100);
            hashMap.put(i1,i1);
        }
    }
}

```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-23/1.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;尝试了很多次,不会发生死循环的情况,但是会出现数据丢失的情况。无论如何,在多线程中涉及到资源竞争的时候不能使用HashMap数据机构,推荐使用ConcurrentHashMap类。

