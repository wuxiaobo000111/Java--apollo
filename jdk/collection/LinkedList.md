### 概述

>在工作中不但要知其然，还要知其所以然。虽然在工作中使用到了LinkedList,也知道其是使用链表的数据结构实现的，
但是对于底层代码却一直是不了解的，所以今天也来分析一下LinkedList的底层原理，帮助更好的使用LindedList。

### UML图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310091713425.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)


>这里提一下一些常见的接口

>Cloneable:实现对象的复制，在LinkedList中使用的是浅复制.

```java
public Object clone() {
        LinkedList<E> clone = superClone();

        // Put clone into "virgin" state
        clone.first = clone.last = null;
        clone.size = 0;
        clone.modCount = 0;

        // Initialize clone with our elements
        for (Node<E> x = first; x != null; x = x.next)
            clone.add(x.item);

        return clone;
    }
```


>Serializable 支持序列化操作

>Deque 可以当做是双端对列

### 属性
```java
  // 节点数据结构，前节点，后节点，和值
  private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
       //链表大小 
      transient int size = 0;
    
        // 指向第一个节点的指针。
        transient Node<E> first;
        //指向最后一个节点的指针。
        transient Node<E> last;
```
### 构造函数
>LinkedList提供了两种构造函数
```java
//可以可到当调用无参构造函数的时候，其实是什么都不做，只有当真正操作这个链表的时候才会进行操作
public LinkedList();


 public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
    
    /**
    *  当添加一个集合的时候，会调用这个方法，首先检验index的值是否合法，如果不合法会抛出越界异常
    *  
    */
    public boolean addAll(int index, Collection<? extends E> c) {
            checkPositionIndex(index);
            Object[] a = c.toArray();
            int numNew = a.length;
            if (numNew == 0)
                return false;
            //找到需要插入的元素的位置
            Node<E> pred, succ;
            if (index == size) {
                succ = null;
                pred = last;
            } else {
                succ = node(index);
                pred = succ.prev;
            }
            // 然后依次加入链表
            for (Object o : a) {
                @SuppressWarnings("unchecked") E e = (E) o;
                Node<E> newNode = new Node<>(pred, e, null);
                //链表为null的情况下
                if (pred == null)
                    first = newNode;
                else
                    pred.next = newNode;
                pred = newNode;
            }
    
            if (succ == null) {
                last = pred;
            } else {
                pred.next = succ;
                succ.prev = pred;
            }
    
            size += numNew;
            modCount++;
            return true;
        }
// checkPositionIndex其实是调用的是isPositionIndex()方法，        
private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

```

### add(E e) 

```java
 public boolean add(E e) {
        linkLast(e);
        return true;
    }
void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

>这个方法就是把这个元素加入到最后一个位置，让原来last的next节点指向newNode，newNode节点的prev节点指向last.

### add(int index, E element)

```java
 public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
// modCount记录的是链表被修改的次数    
void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
    // 我们可以看一下这个方法，这个方法中如果index的大小小于链表大小的一半的时候
    // 会从头部开始查起，否则就会从尾部查起。可以提高效率。
Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```


### clear()
```java
 public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
```

>将链表中的每一个位置都置为空。


### contains(Object o)

```java
public boolean contains(Object o) {
        return indexOf(o) != -1;
    }
    // 循环遍历没有什么好说的
public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }

```



###  descendingIterator()
```java
  /**
     * @since 1.6
     */
    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }
 private class DescendingIterator implements Iterator<E> {
        private final ListItr itr = new ListItr(size());
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
    }
    
    
    private class ListItr implements ListIterator<E> {
            private Node<E> lastReturned;
            private Node<E> next;
            private int nextIndex;
            private int expectedModCount = modCount;
    
            ListItr(int index) {
                // assert isPositionIndex(index);
                next = (index == size) ? null : node(index);
                nextIndex = index;
            }
    
            public boolean hasNext() {
                return nextIndex < size;
            }
    
            public E next() {
                checkForComodification();
                if (!hasNext())
                    throw new NoSuchElementException();
    
                lastReturned = next;
                next = next.next;
                nextIndex++;
                return lastReturned.item;
            }
    
            public boolean hasPrevious() {
                return nextIndex > 0;
            }
    
            public E previous() {
                checkForComodification();
                if (!hasPrevious())
                    throw new NoSuchElementException();
    
                lastReturned = next = (next == null) ? last : next.prev;
                nextIndex--;
                return lastReturned.item;
            }
    
            public int nextIndex() {
                return nextIndex;
            }
    
            public int previousIndex() {
                return nextIndex - 1;
            }
    
            public void remove() {
                checkForComodification();
                if (lastReturned == null)
                    throw new IllegalStateException();
    
                Node<E> lastNext = lastReturned.next;
                unlink(lastReturned);
                if (next == lastReturned)
                    next = lastNext;
                else
                    nextIndex--;
                lastReturned = null;
                expectedModCount++;
            }
    
            public void set(E e) {
                if (lastReturned == null)
                    throw new IllegalStateException();
                checkForComodification();
                lastReturned.item = e;
            }
    
            public void add(E e) {
                checkForComodification();
                lastReturned = null;
                if (next == null)
                    linkLast(e);
                else
                    linkBefore(e, next);
                nextIndex++;
                expectedModCount++;
            }
    
            public void forEachRemaining(Consumer<? super E> action) {
                Objects.requireNonNull(action);
                while (modCount == expectedModCount && nextIndex < size) {
                    action.accept(next.item);
                    lastReturned = next;
                    next = next.next;
                    nextIndex++;
                }
                checkForComodification();
            }
    
            final void checkForComodification() {
                if (modCount != expectedModCount)
                    throw new ConcurrentModificationException();
            }
        }
 // 这个方法是校验链表是否被修改了，如果被修改了则会抛出ConcurrentModificationException异常
 // 这里就涉及到多线程的问题 ，如果遍历的时候，多线程让这个链表加入了一个元素，则这个时候就会抛出异常。
 // 这个地方也证明了LinkedList不是线程安全的。       
 final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```


### element() 

```java
public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }

```

>拿到的是头结点


### get(int index)

```java
// 首先检查是否合法，如果合法调用node这个方法，上面已经讨论个这个问题了，这个就不再重复说明node方法了
public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    
```


### Deque接口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190310130839348.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)

### 
```java
  // 返回头结点的值  
 public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
    
    // 获取尾部节点的值
 public E getLast() {
            final Node<E> l = last;
            if (l == null)
                throw new NoSuchElementException();
            return l.item;
        }
 // 在头部加入一个元素，这个方法可以实现栈的结果       
 public void push(E e) {
             addFirst(e);
         }
 // 返回头部元素，同事移除头部元素        
 public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }   
// 返回头部元素，同事移除头部元素       
public E pop() {
        return removeFirst();
    }    
```
>其实实现什么样的数据结构就看自己的需要，不想过多的描述了。


### LinkedList的遍历方式

```java
 public static void main(String[] args) {
        LinkedList<Integer> linkedList = new LinkedList<>();
        linkedList.push(1);
        linkedList.push(2);
        linkedList.push(3);
        linkedList.push(4);
        // 这里所说的遍历方式是不会删除链表中的元素
        int size = linkedList.size();
        for (int i=0; i<size; i++) {
            System.out.println(linkedList.get(i));
        }

        System.out.println("第二种方式");
        Iterator<Integer> iterator = linkedList.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }


        System.out.println("第三种方式");
        linkedList.stream().forEach(integer -> {
            System.out.println(integer.intValue());
        });


        System.out.println("第四种方式");
        linkedList.parallelStream().forEach(integer -> {
            System.out.println(integer.intValue());
        });


        System.out.println("第五种方式,这个是从尾部输出");
        Iterator<Integer> integerIterator = linkedList.descendingIterator();
        while (integerIterator.hasNext()) {
            System.out.println(integerIterator.next());
        }
    }
```

### ArrayList和LinkedList的区别

>底层数据结构不同，ArrayList是由数组实现的，LinkedList是由双链表实现的。

>由于底层数据结构的不同导致了性能的不同，ArrayList在修改操作的时候，需要进行扩容，在某个索引上加入
元素的时候，需要将之后的元素进行后移，虽然是用System.arraycopy()实现的，但是还是需要移动大量的元素，最坏的打算是需要移动size个元素的位置，
但是查询的时候比较方便，可以通过索引值进行定位。所以ArrayList的查询效率比较高。LinkedList底层是链表，所以在插入或者删除的时候移动的元素比较少，
只是需要移动前后位置的指针，但是同样也是有缺点存在，就是在定位元素的时候，虽然在方法中进行了优化，但是最坏的打算依然是遍历了size/2个长度
的链表大小。所以LinkedList的查询是比较复杂的。所以需要在不同的情况下使用不同的List。

>而且两者都不是线程安全的，ArrayList在添加的时候不是原子操作，LinkedList在每次add的时候都会判断modCount字段
是否合法，所以在涉及到多线程竞争资源的时候不能使用这两类API。