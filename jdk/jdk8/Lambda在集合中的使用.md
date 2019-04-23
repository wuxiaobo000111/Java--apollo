# 流的使用

## 是什么

>&nbsp;&nbsp;&nbsp;&nbsp;流允许声明式方法处理数据集合(通过查询语句来表达),可以认为是遍历数据集的高级迭代器。还可以透明的并行处理。举个小例子:
```java
import java.util.ArrayList;
import java.util.Comparator;

public class Section1 {
    private static ArrayList<Apple> apples = new ArrayList<>() ;
    static {
        Apple apple = new Apple("红色",200);
        apples.add(apple);
        Apple apple1= new Apple("绿色",180);
        apples.add(apple1);
        Apple apple2= new Apple("黄色",140);
        apples.add(apple2);
    }

    public static void main(String[] args) {
        test01();
    }

    public static void test01() {
        apples.stream().sorted(Comparator.comparing(Apple::getWeight))
                .filter(apple -> {
                    return apple.getWeight() > 150;
                }).forEach(apple -> {
            System.out.println(apple.toString());
        });
    }
}

```

>&nbsp;&nbsp;&nbsp;&nbsp;test01方法中就是一个流的例子,stream方法开启了一个流,sorted方法进行排序,filter方法过滤,foreach方法实现了遍历。

>&nbsp;&nbsp;&nbsp;&nbsp;简单定义流就是:"从支持数据处理操作的源生成的元素序列"

```text

1. 元素序列:流提供了一个接口,可以访问特定元素类型的一组有序值。
2.源:提供数据,比如说是集合、数组、输入或者是输出资源.

3.数据处理操作:支持类似于数据库的操作。
```


```java
package com.bobo.basic.jdk8.chapter4;

import com.bobo.basic.jdk8.Dish;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Section2 {

    public static List<Dish> menu = new ArrayList<>();

    static {
        menu = Arrays.asList(
                new Dish("pork",false,800,Dish.Type.MEAT),
                new Dish("beef",false,700,Dish.Type.MEAT),
                new Dish("chickren",false,400,Dish.Type.MEAT),
                new Dish("french fries",true,530,Dish.Type.OTHER),
                new Dish("rice",true,350,Dish.Type.OTHER),
                new Dish("season fruit",true,120,Dish.Type.OTHER),
                new Dish("pizza",true,550,Dish.Type.OTHER),
                new Dish("prawns",false,300,Dish.Type.FISH),
                new Dish("salmon",false,450,Dish.Type.FISH)
        );
    }

    public static void main(String[] args) {
        test01();
    }


    public static void  test01() {
        // 数据源是menu
        // filter,map,limit是数据处理操作
        // menu给流提供了一个元素序列
        List<String> collect = menu.stream().filter(dish -> dish.getCalories() > 300)
                .map(Dish::getName)
                .limit(3)
                .collect(Collectors.toList());
        System.out.println(collect);
    }

}

```

# 流与集合

>&nbsp;&nbsp;&nbsp;&nbsp;java现有的集合和流都提供了接口,来配合代表元素型有序值的接口。<font color="red">注意流只能是消费一次,如果第二次消费就要重新创建一个新的流。

