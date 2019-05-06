```java
package com.bobo.basic.jdk8.chapter7;

import com.bobo.basic.jdk8.Dish;

import java.util.Optional;

public class Optional类 {

    public static void test01 () {
        Dish dish = new Dish("pork", false, 800, Dish.Type.MEAT);
        Optional<Dish> optionalDish = Optional.of(dish);
        if (optionalDish.isPresent()) {
            System.out.println(dish.toString());
        } else {
            System.out.println("dis is not exits");
        }
        optionalDish = Optional.empty();
        if (optionalDish.isPresent()) {
            System.out.println(optionalDish.get().toString());
        } else {
            System.out.println(optionalDish);
        }
    }
    public static void main(String[] args) {
       test01();
    }
}

```