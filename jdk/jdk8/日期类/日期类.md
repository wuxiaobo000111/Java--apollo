```java
package com.bobo.basic.jdk8.date;

import java.time.Duration;
import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.Period;
import java.time.temporal.ChronoField;

/**
 * 关于java8中的日期类:
 * LocalDate, LocalTime,Instant,Duration和Period 最最最简单的使用
 */
public class DateTest {

    /**
     * LocalDate日期类
     */
    public static void test01() {
        LocalDate localDate = LocalDate.of(2019, 4, 30);
        System.out.println(localDate.toString());
        localDate = LocalDate.now();
        System.out.println(localDate.toString());
        //使用TemporalField来去读LocalDate的值
        int year = localDate.get(ChronoField.YEAR);
        System.out.println("year: " + year);
    }

    /**
     * LocalTime的用法
     */
    public static void test02() {
        LocalTime localTime = LocalTime.of(15, 31, 20);
        System.out.println(localTime.toString());
        int hour = localTime.getHour();
        int minute = localTime.getMinute();
        int second = localTime.getSecond();
        System.out.println("hour:" + hour + ", minute: " + minute + ", second: " + second);
        LocalTime parse = LocalTime.parse("15:32:00");
        System.out.println(parse.toString());
    }

    /**
     * LocalDateTime的用法
     */
    public static void test03() {
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime.toString());
        LocalDate localDate = LocalDate.of(2019, 4, 30);
        LocalTime localTime = LocalTime.of(15, 31, 20);
        localDateTime = LocalDateTime.of(localDate, localTime);
        System.out.println(localDateTime.toString());
        System.out.println(localDateTime.toLocalDate().toString() + " " + localDateTime.toLocalTime().toString());
    }

    /**
     * Instant的用法
     */
    public static void test04() {
        //从1970年1月1日之后加上
        Instant instant = Instant.ofEpochSecond(3);
        System.out.println(instant.toString());
        //注意这里会抛出异常信息
        int i = Instant.now().get(ChronoField.DAY_OF_MONTH);
        System.out.println(i);
    }

    /**
     * Duration的用法:主要用于以秒或者是纳秒来衡量时间的长短。如果需要比较日期，则可以使用Period类
     */
    public static void test05() {
        Duration between = Duration.between(LocalTime.of(15, 46, 0), LocalTime.now());
        System.out.println(between.getSeconds());
        Period period = Period.between(LocalDate.of(2018, 4, 30), LocalDate.of(2019, 3, 29));
        System.out.println(period.getYears());
        System.out.println(period.getMonths());
        System.out.println(period.getDays());
    }

    public static void main(String[] args) {
        test05();
    }
}
```