# JDK1.8中的Spliterator

# 定义

>&nbsp;&nbsp;&nbsp;&nbsp;用于遍历和分割“源”元素的对象。

# 数据源

>&nbsp;&nbsp;&nbsp;&nbsp;Spliterator的元素来源可能是一个数组，一个集合，一个IO通道，一个生成函数。

# 处理数据源的方式

>&nbsp;&nbsp;&nbsp;&nbsp;   Spliterator可以单独或顺序地批量地遍历元素。
>&nbsp;&nbsp;&nbsp;&nbsp;   Spliterator也可以将其部分元素作为另一个Spliterator进行分区，为了并行化操作。使用不能拆分或以非常不平衡或低效的方式进行拆分Spliterator的操作不太可能从并行中获益。遍历和分解流出的元素;每个Spliterator只对单个批量计算有用。

# 例子(直接上代码看问题)

```java
package com.bobo.basic.collection;

import java.util.ArrayList;
import java.util.List;
import java.util.Spliterator;


public class SpliteratorTest {

    public static void main(String[] args) {

        ArrayList<Integer> list = new ArrayList<>(20);
        for (int i=1; i<= 20; i++) {
            list.add(i);
        }

        ////spliterator有20个元素
        Spliterator<Integer> spliterator = list.spliterator();
        //integerSpliterator1中有10个,spliterator中有10个
        Spliterator<Integer> integerSpliterator1 = spliterator.trySplit();
        // integerSpliterator2中有5个,integerSpliterator1中有五个
        Spliterator<Integer> integerSpliterator2 = integerSpliterator1.trySplit();
        // integerSpliterator3中有5个,spliterator中有5个
        Spliterator<Integer> integerSpliterator3 = spliterator.trySplit();

        MyThread myThread1 = new MyThread(spliterator);
        MyThread myThread2 = new MyThread(integerSpliterator1);
        MyThread myThread3 = new MyThread(integerSpliterator2);
        MyThread myThread4 = new MyThread(integerSpliterator3);

        myThread1.setName("thread1");
        myThread2.setName("thread2");
        myThread3.setName("thread3");
        myThread4.setName("thread4");


        myThread1.start();
        myThread2.start();
        myThread3.start();
        myThread4.start();


    }


}

class MyThread extends Thread {

    private Spliterator<Integer> spliterator;

    public MyThread (Spliterator<Integer> spliterator) {
        this.spliterator = spliterator;
    }
    @Override
    public void run() {
        spliterator.forEachRemaining(integer -> {
            System.out.println(Thread.currentThread().getName()+"------"+integer);
        });
    }
}

```

>&nbsp;&nbsp;&nbsp;&nbsp;这里只是简单的写了一个小例子。下面我就根据ArrayList中的ArrayListSpliterator来分析一波源代码.


```java
    static final class ArrayListSpliterator<E> implements Spliterator<E> {

        //分割后List集合对象
        private final ArrayList<E> list;
        //起始位置
        private int index; // current index, modified on advance/split
        //结束位置，如果是-1则表示是到最后一个元素
        private int fence; // -1 until used; then one past last index
        //这个是用于多线程中的判断,初始值就是ArrayList本身记录的被修改的次数。
        private int expectedModCount; // initialized when fence set

        /**构造函数，不用多说 */
        ArrayListSpliterator(ArrayList<E> list, int origin, int fence,
                             int expectedModCount) {
            this.list = list; // OK if null unless traversed
            this.index = origin;
            this.fence = fence;
            this.expectedModCount = expectedModCount;
        }
        //拿到结束位置的值。
        private int getFence() { // initialize fence to size on first use
            int hi; // (a specialized variant appears in method forEach)
            ArrayList<E> lst;
            if ((hi = fence) < 0) {
                if ((lst = list) == null)
                    hi = fence = 0;
                else {
                    expectedModCount = lst.modCount;
                    hi = fence = lst.size;
                }
            }
            return hi;
        }

        //对现有的再次进行分割，可以看到每次都分割为原来的一半大小,从上述中的例子也可以看出。
        public ArrayListSpliterator<E> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null : // divide range in half unless too small
                new ArrayListSpliterator<E>(list, lo, index = mid,
                                            expectedModCount);
        }

        //获取头部的值，然后进行操作,同时将起始位置+1,可以认为///是删除了头部元素,当还有未处理元素的时候会返回true,如///果返回了false,则表示所有的元素都处理完毕
        public boolean tryAdvance(Consumer<? super E> action) {
            if (action == null)
                throw new NullPointerException();
            int hi = getFence(), i = index;
            if (i < hi) {
                index = i + 1;
                @SuppressWarnings("unchecked") E e = (E)list.elementData[i];
                action.accept(e);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            return false;
        }

        //遍历List中所有的元素
        public void forEachRemaining(Consumer<? super E> action) {
            int i, hi, mc; // hoist accesses and checks from loop
            ArrayList<E> lst; Object[] a;
            if (action == null)
                throw new NullPointerException();
            //当还有元素的时候    
            if ((lst = list) != null && (a = lst.elementData) != null) {
                //如果结束为止小于0,表示是到结束位置
                if ((hi = fence) < 0) {
                    mc = lst.modCount;
                    hi = lst.size;
                }
                else
                    mc = expectedModCount;
                //如果还有元素的时候,
                if ((i = index) >= 0 && (index = hi) <= a.length) {
                    for (; i < hi; ++i) {
                        @SuppressWarnings("unchecked") E e = (E) a[i];
                        action.accept(e);
                    }
                    if (lst.modCount == mc)
                        return;
                }
            }
            throw new ConcurrentModificationException();
        }

        public long estimateSize() {
            //返回还未处理的长度
            return (long) (getFence() - index);
        }


        //特征值
        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }
```

# 补充特征值
```java
ORDERED int 型  值为16  既定的顺序，Spliterator保证拆分和遍历时是按照这一顺序。
DISTINCT int型 值为1 表示元素都不是重复的，对于每一对元素{ x,  y}，{ !x.equals(y)}。例如，这适用于基于{@link Set}的Spliterator。
SORTED int型 值为4 表示元素顺序按照预定义的顺序，可以通过getComparator 获取排序器，若返回null ,则是按自然排序。
SIZED int型 值为64 表示在遍历分隔之前 estimateSize() 返回的值代表一个有限的大小，在没有
修改结构源的情况下，代表了一个完整遍历时所遇到的元素数量的精确计数。
NONNULL init型 值为256 表示数据源保证元素不会为空
IMMUTABLE int 型 值为1024 表示在遍历的过程中不能添加、替换、删除元素
CONCURRENT int型 值为4096 表示元素可以被多个线程安全并发得修改而不需要外部的同步。
SUBSIZED int型 值为16384 表示trySplit()返回的结果都是SIZED和SUBSIZED

```

