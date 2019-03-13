### ArrayList源码分析

### ArrayList的成员变量

>对于大多数人来说，大家都知道ArrayList的底层是用数组实现的，那么今天就来揭秘一下ArrayList的底层的实现原理。首先来看一下
ArrayList的成员变量

```java
/**
     * Default initial capacity.
     */
    // 默认大小
    private static final int DEFAULT_CAPACITY = 10;

    // null数组，之后解释是干嘛的
    private static final Object[] EMPTY_ELEMENTDATA = {};

    // 默认是null数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    // ArrayList真正意义上数组保存在这个数组中
    transient Object[] elementData; // non-private to simplify nested class access

    // 数组大小
    private int size;
    
    // List对象有一个成员变量modCount，它代表该List对象被修改的次数，每对List对象修改一次，modCount都会加1.
     protected transient int modCount = 0;

```

### ArrayList的构造函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190309211000288.)
>可以看到以一共有三个构造函数。
```java
// 指定大小，当参数合法的时候会初始化elementData数组，如果是0则是null数组
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

// 不指定就是空数组
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

   
// 如果参数是一个Collection的子类，则会调用Arrays.copyOf()方法    
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

### 常见函数解析

#### add()

```java
  //在数组尾部插入操作，首先是进行扩容操作
  public boolean add(E e) {
         ensureCapacityInternal(size + 1);  // Increments modCount!!
         elementData[size++] = e;
         return true;
     }


    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    //判断一个数组是否是空数组，如果是则返回默认大小(10)和传入参数的最大值，
    //如果不是,则返回传入的大小
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
    //如果说传入的大小比现在数组长度要长，则会进行扩容操作。比如现在有10个元素，
    //则size的大小是10，elementData.length也是10，当我想再加入一个新的元素的时候
    //则发现这个时候长度不够了，则会进行扩容操作，即调用grow()方法这个时候minCapacity的大小是11
     private void ensureExplicitCapacity(int minCapacity) {
            modCount++;
    
            // overflow-conscious code
            if (minCapacity - elementData.length > 0)
                grow(minCapacity);
    }
    
     private void grow(int minCapacity) {
            // overflow-conscious code
            int oldCapacity = elementData.length;
            //这个地方我们可以看到是oldCapacity右移了1个单位。比如原来是2，
            // 2的二进制是10，右边移动是01,索引新的大小是原来大小的1.5倍
            int newCapacity = oldCapacity + (oldCapacity >> 1);
            if (newCapacity - minCapacity < 0)
                newCapacity = minCapacity;
            if (newCapacity - MAX_ARRAY_SIZE > 0)
                newCapacity = hugeCapacity(minCapacity);
            // minCapacity is usually close to size, so this is a win:
            // 然后就会进行数组的拷贝
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
     //在底层调用的其实是 System.arraycopy(original, 0, copy, 0,
                                         Math.min(original.length, newLength));这个方法，这个方法其实是个native方法。
                                  
```
```text
public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
代码解释:
　　Object src : 原数组
    int srcPos : 从元数据的起始位置开始
　　Object dest : 目标数组
　　int destPos : 目标数组的开始起始位置
　　int length  : 要copy的数组的长度
```



### 扩展 ArrayList不是线程安全的
```java
elementData[size++] = e;
// 这个操作不是原子操作

单线程执行这段代码完全没问题，可是到多线程环境下可能就有问题了。可能一个线程会覆盖另一个线程的值。

列表为空 size = 0。
线程 A 执行完 elementData[size] = e;之后挂起。A 把 "a" 放在了下标为 0 的位置。此时 size = 0。
线程 B 执行 elementData[size] = e; 因为此时 size = 0，所以 B 把 "b" 放在了下标为 0 的位置，于
是刚好把 A 的数据给覆盖掉了。
线程 B 将 size 的值增加为 1。
线程 A 将 size 的值增加为 2。

这样子，当线程 A 和线程 B 都执行完之后理想情况下应该是 "a" 在下标为 0 的位置，"b" 在标为 1 的位置。而
实际情况确是下标为 0 的位置为 "b"，下标为 1 的位置啥也没有。




ArrayList 默认数组大小为 10。假设现在已经添加进去 9 个元素了，size = 9。

线程 A 执行完 add 函数中的ensureCapacityInternal(size + 1)挂起了。
线程 B 开始执行，校验数组容量发现不需要扩容。于是把 "b" 放在了下标为 9 的位置，且 size 自增 1。此时 size = 10。
线程 A 接着执行，尝试把 "a" 放在下标为 10 的位置，因为 size = 10。但因为数组还没有扩容，最大的下标才为 
9，所以会抛出数组越界异常 ArrayIndexOutOfBoundsException


```




### add(int index, E element)

>在指定的索引位置进行插入操作


```java
public void add(int index, E element) {
        // 首先校验索引是否超过了真实数组的长度，这里指的是已经存放的元素的个数
        // 如果超过了则会报异常
        rangeCheckForAdd(index);
        // 这个上面已经提到过了
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 这个时候index位置之后的元素会向后移动一位，把index上这个位置空出来
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        // 在index元素上进行赋值操作
        elementData[index] = element;
        // 真实大小自增
        size++;
    }
    
    private void rangeCheckForAdd(int index) {
            if (index > size || index < 0)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190309214505743.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)


###  addAll(int index, Collection<? extends E> c)

```java
 public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

```
>这里可以看出其实和add(int index, E element)这个方法一致，只不过是这个方法是空出来了collection长度个位置，然后调用了
System.arraycopy进行填充。



### remove()
```java
 public E remove(int index) {
        //检查索引值是否越界。如果越界则会抛出IndexOutOfBoundsException异常
        rangeCheck(index);

        modCount++;
        通过索引值获取元素
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

```

### removeAll(Collection<?> c)

```java
private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        // 循环遍历传入的collection，然后将collection中的元素首先放在
        // elementData数组中
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```

>这个方法的算法思想就是:首先找到当前数组中不包含的元素放在头部，然后将尾部需要删除的元素的长度上的元素都置为null。



### 外传一------System.arraycopy()的分析

>System中提供了一个native静态方法arraycopy(),可以使用这个方法来实现数组之间的复制。对于一维数组来说，这种复制属性值传递，修改副本不会影响原来的值。对于二维或者一维数组中存放的是对象时，复制结果是一维的引用变量传递给副本的一维数组，修改副本时，会影响原来的数组。


### 外传二-------自己实现的List结构
```java
package com.bobo.dataStructure.list;

/**
 * 线性表
 **/
public class MyList {
    private int[] array;

    private int size=0;

    private static final int DEFAULT_ARRAY_LENGTH =8;

    public MyList(int length) {
        array = new int[length];
    }
    public MyList() {
        array = new int[DEFAULT_ARRAY_LENGTH];
    }

    /**
     * 清空线性表
     */
    public void clearList()  {
        array = new int[0];
        size=0;
    }

    /**
     * 返回线性表的元素个数，这个不是线性表的长度
     * @return
     */
    public int listLength() {
        return size;
    }

    /**
     * 返回索引值为i的元素
     * @param i
     * @return
     */
    public int getElement(int i) {
        if (i < 0 || i > array.length) {
            throw new RuntimeException("数组越界异常");
        }
        return array[i];
    }

    /**
     * 判断线性表中是否存在这个元素，如果存在，则返回
     * 在数组中的位置，如果不存在则直接返回0
     * @param element
     * @return
     */
    public int locateElement(int element) {
        for (int i=0; i< array.length; i++) {
            if (array[i]==element) {
                return i+1;
            }
        }
        return 0;
    }

    /**
     * insert(i,e)将元素e插入到线性表i中的位置
     * 如果position超过现在数组的长度，则扩展到position*2个数组的大小
     * 然后再进行插入
     * 1,2,3,4
     */

    public void insert(int position, int newElement) {
        if (position <= 0) {
            throw new RuntimeException("索引值参数错误");
        }

        if (position >= array.length) {
            expandArray(position*2);
            array[position-1]=newElement;
        } else{
            for (int i= array.length;i>position;i--){
                array[i-1]= array[i-2];
            }
            array[position-1]=newElement;
        }
        size++;
    }

    /**
     * 对数组大小进行扩容
     * @param size
     */
    private void expandArray(int size) {
        int[] newArray = new  int[size];
        for (int i=0;i<array.length;i++) {
            newArray[i]= array[i];
        }
        array = newArray;
    }

    /**
     * 打印出整个数据结构
     */
    public void printList() {
        for (int i =0;i<array.length;i++) {
            System.out.print(array[i] +" ");
        }
        System.out.println("");
    }

    /**
     * 在existElement前插入一个新的元素
     * @param existElement
     * @param newElement
     */
    public void insertBefore(int existElement, int newElement) {
        //首先定位已经存在的元素的位置
        int index = this.locateElement(existElement);
        if (index == 0) {
            throw new RuntimeException("existElement:"+existElement+"不存在");
        }
        insert(index,newElement);
    }

    /**
     * remove(i)删错线性表中索引为i的元素，并返回值
     * @param index
     */
    public int remove(int index) {
        if (index <0) {
            throw new RuntimeException("索引值不能为负值");
        }
        if(index > array.length-1) {
            throw new IndexOutOfBoundsException();
        }
        index = index-1;
        int temp = array[index];
        for (int i=index;i<array.length-1;i++) {
            array[i]= array[i+1];
        }
        size--;
        return temp;
    }

    /**
     * remove(e)删错线性表中e元素，成功返回true；
     * @param element
     */

    public boolean removeElement(int element) {
        while (true) {
            int index = locateElement(element);
            if(index == 0) {
                break;
            }
            remove(index);
        }
        return true;
    }

    /**
     * replace(i,e)替换线性表中索引为i的数据元素为e，返回原数据元素，如果i越界则报错；
     * 应该是把校验数据的代码拿出来
     * @param index 索引值
     * @param element 新元素值
     */
    public int replace(int index, int element) {
        if (index <0) {
            throw new RuntimeException("索引值不能为负值");
        }
        if(index > array.length-1) {
            throw new IndexOutOfBoundsException();
        }
        int temp = array[index-1];
        array[index-1] = element;
        return temp;
    }

    public static void main(String[] args) {
        MyList myList = new MyList();
        myList.insert(1,1);
        myList.insert(2,1);
        myList.insert(3,3);
        myList.printList();
        myList.removeElement(1);
        myList.printList();

      /*  myList.insert(1,1);
        myList.printList();
        myList.insert(10,2);
        myList.printList();
        myList.insertBefore(2,3);
        myList.printList();*/
    }
}

```