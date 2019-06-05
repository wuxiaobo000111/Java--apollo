>摘要: 原创出处 http://cmsblogs.com/?p=2371 「小明哥」欢迎转载，保留摘要，谢谢！

# SkipList

>&nbsp;&nbsp;&nbsp;&nbsp;什么是SkipList？Skip List ，称之为跳表，它是一种可以替代平衡树的数据结构，其数据元素默认按照key值升序，天然有序。Skip list让已排序的数据分布在多层链表中，以0-1随机数决定一个数据的向上攀升与否，通过“空间来换取时间”的一个算法，在每个节点中增加了向前的指针，在插入、删除、查找时可以忽略一些不可能涉及到的结点，从而提高了效率。

>&nbsp;&nbsp;&nbsp;&nbsp;我们先看一个简单的链表，如下：
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/69.png?raw=true)


>&nbsp;&nbsp;&nbsp;如果我们需要查询9、21、30，则需要比较次数为3 + 6 + 8 = 17 次，那么有没有优化方案呢？有！我们将该链表中的某些元素提炼出来作为一个比较“索引”，如下：

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/70.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;我们先与这些索引进行比较来决定下一个元素是往右还是下走，由于存在“索引”的缘故，导致在检索的时候会大大减少比较的次数。当然元素不是很多，很难体现出优势，当元素足够多的时候，这种索引结构就会大显身手。

## skiplist的特性

```text
1. 由很多层结构组成，level是通过一定的概率随机产生的

2. 每一层都是一个有序的链表，默认是升序，也可以根据创建映射时所提供的Comparator进行排序，具体取决于使用的
构造方法

3. 最底层(Level 1)的链表包含所有元素

4. 如果一个元素出现在Level i 的链表中，则它在Level i 之下的链表也都会出现

5. 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/71.png?raw=true)

## skiplist的查找

>&nbsp;&nbsp;&nbsp;&nbsp;SkipListd的查找算法较为简单，对于上面我们我们要查找元素21，其过程如下：

```text
比较3，大于，往后找（9），

比9大，继续往后找（25），但是比25小，则从9的下一层开始找（16）

16的后面节点依然为25，则继续从16的下一层找

找到21
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/72.png?raw=true)

## skiplist的插入

>&nbsp;&nbsp;&nbsp;&nbsp;SkipList的插入操作主要包括：

```text
    查找合适的位置。这里需要明确一点就是在确认新节点要占据的层次K时，采用丢硬币的方式，完全随机。如果
    占据的层次K大于链表的层次，则重新申请新的层，否则插入指定层次

    申请新的节点

    调整指针

```
>&nbsp;&nbsp;&nbsp;&nbsp;假定我们要插入的元素为23，经过查找可以确认她是位于21后，9、16、21前。当然需要考虑申请的层次K。
&nbsp;&nbsp;&nbsp;&nbsp;如果层次K > 3,需要申请新层次（Level 4）

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/74.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;如果层次 K = 2,直接在Level 2 层插入即可

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/75.png?raw=true)


## SkipList的删除
>&nbsp;&nbsp;&nbsp;&nbsp;删除节点和插入节点思路基本一致：找到节点，删除节点，调整指针。比如删除节点9，如下：


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/76.png?raw=true)


# ConcurrentSkipListMap

>&nbsp;&nbsp;&nbsp;&nbsp;通过上面我们知道SkipList采用空间换时间的算法，其插入和查找的效率O(logn)，其效率不低于红黑树，但是其原理和实现的复杂度要比红黑树简单多了。一般来说会操作链表List，就会对SkipList毫无压力。
&nbsp;&nbsp;&nbsp;&nbsp;ConcurrentSkipListMap其内部采用SkipLis数据结构实现。为了实现SkipList，ConcurrentSkipListMap提供了三个内部类来构建这样的链表结构：Node、Index、HeadIndex。其中Node表示最底层的单链表有序节点、Index表示为基于Node的索引层，HeadIndex用来维护索引层次。到这里我们可以这样说ConcurrentSkipListMap是通过HeadIndex维护索引层次，通过Index从最上层开始往下层查找，一步一步缩小查询范围，最后到达最底层Node时，就只需要比较很小一部分数据了。在JDK中的关系如下图：


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/77.png?raw=true)


```java
   static final class Node<K,V> {
        final K key;
        volatile Object value;
        volatile Node<K,V> next;
   }
    static class Index<K,V> {
        final Node<K,V> node;
        final Index<K,V> down;
        volatile Index<K,V> right;
    }

   static final class HeadIndex<K,V> extends Index<K,V> {
        final int level;
        HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
            super(node, down, right);
            this.level = level;
        }
    }  
```