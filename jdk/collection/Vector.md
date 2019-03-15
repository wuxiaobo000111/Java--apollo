### 概述

>&nbsp;&nbsp;&nbsp;&nbsp;正如我们大家都知道的样子，在集合框架中ArrayList和LinkedList都是线程不安全的。但是有没有存在线程安全的类呢？答案必然是肯定的。这里就解释一下Vector这个类。可能有人会提到java.util.concurrent.CopyOnWriteArrayList这个东西，由于其实现机制和Vector是不同的，这里不做如何讨论。当分析到CopyOnWriteArrayList这个类的时候我会详细介绍一下这个东东。



>&nbsp;&nbsp;&nbsp;&nbsp;首先我们来看一下如下所示的代码

```java
 public static void main(String[] args) throws InterruptedException {
        Vector<Integer> vector = new Vector<>();
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10,20,10L,TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(100));

        for (int i=0; i<20; i++) {
            int temp = i;
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    vector.add(temp);
                }
            });
        }
        // 这里使用sleep就是让main线程阻塞，让线程池中的vector调用完成。
        Thread.sleep(2000);
        System.out.println(vector.toString());
    }
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/1.jpg?raw=true)