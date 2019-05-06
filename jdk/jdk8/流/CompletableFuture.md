```java
package com.bobo.basic.jdk8.chapter11;

public class DelayUtil {

    public static void  delay () {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

package com.bobo.basic.jdk8.chapter11;

import java.util.Random;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.Future;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Shop {

    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public Shop() {

    }
    public Shop(String name) {
        this.name = name;
    }

    public static void main(String[] args) throws Exception {
        Shop shop = new Shop("BestShop");
        Future<Double> myFavoriteProduct = shop.getPriceAsync("my favorite product");
        // 这个get注意:要么获取到返回值,如果没有返回值的话就会在一直阻塞中。
        Double aDouble = myFavoriteProduct.get();
        System.out.println("price: "+ aDouble);
    }
    public double getPrice (String product) {
        return calculatePrice(product);
    }

    public Future<Double> getPriceAsync (String product) {
        CompletableFuture<Double> future = new CompletableFuture<>();
        new Thread(() ->{
            try {
                double calculatePrice = calculatePrice(product);
                future.complete(calculatePrice);
            }catch (Exception e) {
                //异常处理
                future.completeExceptionally(e);
            }
        }).start();

        ReentrantLock lock = new ReentrantLock();
        lock.lock();
        return future;
    }
    private double calculatePrice(String product) {
        DelayUtil.delay();
        Random random = new Random();
        return random.nextDouble() * product.charAt(0) + product.charAt(1);
    }



}


```