
--------------------- 
作者：octopusflying 
来源：CSDN 
原文：https://blog.csdn.net/octopusflying/article/details/80634864 
版权声明：本文为博主原创文章，转载请附上博文链接！

摘要: 原创出处 http://cmsblogs.com/?p=2269 「小明哥」欢迎转载，保留摘要，谢谢！
# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;   java.util.concurrent包中的Exchanger类可用于两个线程之间交换信息。可简单地将Exchanger对象理解为一个包含两个格子的容器，通过exchanger方法可以向两个格子中填充信息。当两个格子中的均被填充时，该对象会自动将两个格子的信息交换，然后返回给线程，从而实现两个线程的信息交换。
&nbsp;&nbsp;&nbsp;&nbsp;另外需要注意的是，Exchanger类仅可用作两个线程的信息交换，当超过两个线程调用同一个exchanger对象时，得到的结果是随机的，exchanger对象仅关心其包含的两个“格子”是否已被填充数据，当两个格子都填充数据完成时，该对象就认为线程之间已经配对成功，然后开始执行数据交换操作。


# 示例代码

```java
package com.bobo.basic.thread;


import java.util.concurrent.Exchanger;
import java.util.concurrent.Semaphore;


class ExchangerTest extends Thread {
    private Exchanger<String> exchanger;
    private String string;
    private String threadName;

    public ExchangerTest(Exchanger<String> exchanger, String string,
                         String threadName) {
        this.exchanger = exchanger;
        this.string = string;
        this.threadName = threadName;
    }

    @Override
    public void run() {
        try {
            System.out.println(threadName + ": " + exchanger.exchange(string));
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

public class Test {
    public static void main(String[] args) {
        Exchanger<String> exchanger = new Exchanger<>();
        ExchangerTest test1 = new ExchangerTest(exchanger, "string1",
                "thread-1");
        ExchangerTest test2 = new ExchangerTest(exchanger, "string2",
                "thread-2");

        test1.start();
        test2.start();
    }
}
```
>额,原理我就不写了,表示我太菜,有很多地方都看不懂。