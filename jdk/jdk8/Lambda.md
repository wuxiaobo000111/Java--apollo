# Lambda表达式的三部分

```java
1. 参数列表
2. 箭头:把参数列表和Lambda主体分割开。
3. Lambda

  filterGreenApples(apples,apple -> "green".equals(apple.getColor()));

  其中,apple就是参数,"green".equals(apple.getColor())是Lambda主体。
```

# 函数式接口

>&nbsp;&nbsp;&nbsp;&nbsp;只是定义一个抽象方法的接口。