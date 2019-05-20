原文出处：http://cmsblogs.com/ 『chenssy』

# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;在执行程序时，为了提供性能，处理器和编译器常常会对指令进行重排序，但是不能随意重排序，不是你想怎么排序就怎么排序，它需要满足以下两个条件：在单线程环境下不能改变程序运行的结果;存在数据依赖关系的不允许重排序。


# as-if-serial语义

>&nbsp;&nbsp;&nbsp;&nbsp;as-if-serial语义的意思是，所有的操作均可以为了优化而被重排序，但是你必须要保证重排序后执行的结果不能被改变，编译器、runtime、处理器都必须遵守as-if-serial语义。注意as-if-serial只保证单线程环境，多线程环境下无效。下面我们用一个简单的示例来说明：

```java
下面我们用一个简单的示例来说明：

int a = 1 ;      //A
int b = 2 ;      //B
int c = a + b;   //C
```

>&nbsp;&nbsp;&nbsp;&nbsp;A、B、C三个操作存在如下关系：A、B不存在数据依赖关系，A和C、B和C存在数据依赖关系，因此在进行重排序的时候，A、B可以随意排序，但是必须位于C的前面，执行顺序可以是A –> B –> C或者B –> A –> C。但是无论是何种执行顺序最终的结果C总是等于3。as-if-serail语义把单线程程序保护起来了，它可以保证在重排序的前提下程序的最终结果始终都是一致的。
&nbsp;&nbsp;&nbsp;&nbsp;1、2是程序顺序次序规则，3是传递性。但是，不是说通过重排序，B可能会排在A之前执行么，为何还会存在存在A happens-beforeB呢？这里再次申明A happens-before B不是A一定会在B之前执行，而是A的对B可见，但是相对于这个程序A的执行结果不需要对B可见，且他们重排序后不会影响结果，所以JMM不会认为这种重排序非法。
&nbsp;&nbsp;&nbsp;&nbsp;下面我们在看一段有意思的代码：

```java
public class RecordExample1 {
    public static void main(String[] args){
        int a = 1;
        int b = 2;

        try {
            a = 3;           //A
            b = 1 / 0;       //B
        } catch (Exception e) {

        } finally {
            System.out.println("a = " + a);
        }
    }
}
```
>&nbsp;&nbsp;&nbsp;&nbsp;按照重排序的规则，操作A与操作B有可能会进行重排序，如果重排序了，B会抛出异常（ / by zero），此时A语句一定会执行不到，那么a还会等于3么？如果按照as-if-serial原则它就改变了程序的结果。其实JVM对异常做了一种特殊的处理，为了保证as-if-serial语义，Java异常处理机制对重排序做了一种特殊的处理：JIT在重排序时会在catch语句中插入错误代偿代码（a = 3）,这样做虽然会导致cathc里面的逻辑变得复杂，但是JIT优化原则是：尽可能地优化程序正常运行下的逻辑，哪怕以catch块逻辑变得复杂为代价。

>&nbsp;&nbsp;&nbsp;&nbsp;重排序不会影响单线程环境的执行结果，但是会破坏多线程的执行语义。