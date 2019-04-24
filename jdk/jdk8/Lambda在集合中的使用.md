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

# 流的终端操作


# 使用流

```text
1. 一个数据源(集合)来执行一个查询。
2. 一个中间操作链,形成一条流的流水线。
3. 一个终端操作,执行流水线,并且生成结果。
```

```java
package com.bobo.basic.jdk8.chapter5;

import com.alibaba.fastjson.JSONObject;
import com.bobo.basic.jdk8.Dish;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class Section1 {

    public static List<Dish> menu = new ArrayList<>();

    static {
        menu = Arrays.asList(
                new Dish("pork", false, 800, Dish.Type.MEAT),
                new Dish("beef", false, 700, Dish.Type.MEAT),
                new Dish("chickren", false, 400, Dish.Type.MEAT),
                new Dish("french fries", true, 530, Dish.Type.OTHER),
                new Dish("rice", true, 350, Dish.Type.OTHER),
                new Dish("season fruit", true, 120, Dish.Type.OTHER),
                new Dish("pizza", true, 550, Dish.Type.OTHER),
                new Dish("prawns", false, 300, Dish.Type.FISH),
                new Dish("salmon", false, 450, Dish.Type.FISH)
        );
    }


    /**
     * 流中的filter方法
     */
    public static void test01() {
        List<Dish> collect = menu.stream().filter(Dish::isVegetarian).collect(Collectors.toList());
        System.out.println(collect.toString());
    }


    /**
     * distinct 去重
     */
    public static void test02() {
        List<Integer> list = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
        list.stream().filter(integer -> integer % 2 == 0)
                .distinct().forEach(integer -> {
            System.out.println(integer.toString());
        });
    }


    /**
     * 截断
     */
    public static void test03() {
        List<Dish> collect = menu.stream().filter(dish -> dish.getCalories() > 300)
                .limit(3).collect(Collectors.toList());
        System.out.println(collect.toString());
    }


    /**
     * 跳过
     */
    public static void test04() {
        List<Dish> collect = menu.stream().filter(dish -> dish.getCalories() > 300)
                .skip(2).collect(Collectors.toList());
        System.out.println(collect.toString());
    }


    /**
     * 对流中的每一个元素应用函数: 接受一个函数作为参数,然后应用到每个元素上,并将其映射为一个新的元素,
     */
    public static void test05() {
        List<Integer> collect = menu.stream().map(Dish::getName).map(String::length).collect(Collectors.toList());
        System.out.println(collect.toString());
    }


    /**
     * 流的扁平化:
     * flatMap方法的效果是:各个数组并不是分别映射成为一个流,而是映射成为流的内容,所以使用flatMap(Arrays::stream)
     * 的时候是把生成的流都合并起来,扁平化一个流。
     */
    public static void test06() {
        List<String> strings = Arrays.asList("Hello", "World");
        List<String> collect = strings.stream().map(s -> s.split("")).flatMap(Arrays::stream).distinct().collect(Collectors.toList());
        System.out.println(JSONObject.toJSONString(collect));
    }


    /**
     * 是否至少匹配一个元素:
     * <p>
     * 流中能否有一个元素能匹配给定的谓词
     */
    public static void test07() {
        if (menu.stream().anyMatch(Dish::isVegetarian)) {
            System.out.println("The menu is (somewhat) vegetarian friendly");
        }
    }


    /**
     * 所有的都匹配
     */
    public static void test08() {
        if (menu.stream().anyMatch(dish -> dish.getCalories() < 1000)) {
            System.out.println("菜谱的所有的热量都小于1000");
        }
    }


    /**
     * 所有的都不匹配
     */
    public static void test09() {
        if (menu.stream().noneMatch(dish -> dish.getCalories() > 1000)) {
            System.out.println("菜谱的所有的热量都小于1000");
        }
    }

    /**
     * findAny方法返回的是当前流中的任意元素:
     * 流水线在后台进行优化使其只需要走一遍,在利用短路找到结果的时候立即结束
     * Optional是一个容器类,代表一个值存在或者不存在
     */
    public static void test10() {
        Optional<Dish> any = menu.stream().filter(Dish::isVegetarian).findAny();
        if (any.isPresent()) {
            System.out.println("有值");
        }
        /**
         * 当有值存在的时候会执行指定的代码块
         */
        any.ifPresent(dish -> {
            System.out.println(dish.getName());
        });

        //当有值的时候就会返回
        Dish dish = any.get();
    }


    /**
     * 查找第一个元素
     */
    public static void test11() {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
        Optional<Integer> first = list.stream().map(integer -> integer * integer).filter(integer -> integer % 3 == 0).findFirst();
        if (first.isPresent()) {
            System.out.println("exits");
        } else {
            System.out.println("no exits");
        }
    }


    /**
     * reduce 方法两个参数
     *      第一个参数是初始值。
     *      第二个参数 BinaryOperator<T> accumulator 将两个元素组合起来产生一个新值
     */

    public static void test12() {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
        Integer reduce = list.stream().reduce(0, (a, b) -> a + b);
        System.out.println(reduce);

        Optional<Integer> reduce1 = list.stream().reduce(Integer::max);
        if (reduce1.isPresent()) {
            System.out.println(reduce1.get());
        }
    }


    public static void main(String[] args) {
        test12();
    }
}

```

# 数值流

>&nbsp;&nbsp;&nbsp;&nbsp;stream api中提供了原始类型流特化,专门支持处理数值流的方法
## 原始类型流特化

>&nbsp;&nbsp;&nbsp;&nbsp;jdk8提供了三个原始类型特化流接口来解决这个问题:intStream,DoubleStream和LongStream,分别将流中的元素转化为int,long和double。避免暗含的装箱成本。