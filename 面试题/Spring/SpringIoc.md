# Spring Ioc的实现

<a name="P4qwi"></a>
# 什么是Spring Ioc
Spring 框架的核心是 Spring IoC 容器。容器创建 Bean 对象，将它们装配在一起，配置它们并管理它们的完整生命周期。

- Spring 容器使用**依赖注入**来管理组成应用程序的 Bean 对象。
- 容器通过读取提供的**配置元数据** Bean Definition 来接收对象进行实例化，配置和组装的指令。
- 该配置元数据 Bean Definition 可以通过 XML，Java 注解或 Java Config 代码**提供**。

![](https://cdn.nlark.com/yuque/0/2019/jpeg/382109/1561453353533-0da386fd-5fd8-42e2-b54d-3b1ffcdd49cd.jpeg#align=left&display=inline&height=252&originHeight=252&originWidth=300&size=0&status=done&width=300)

<a name="Uf4U5"></a>
# 什么是依赖注入
在依赖注入中，你不必主动、手动创建对象，但必须描述如何创建它们。

- 你不是直接在代码中将组件和服务连接在一起，而是描述配置文件中哪些组件需要哪些服务。
- 然后，再由 IoC 容器将它们装配在一起。

<a name="0o64l"></a>
# Ioc和DI（依赖注入有什么区别）
IoC 是个更宽泛的概念，DI 是更具体的。

![](https://cdn.nlark.com/yuque/0/2019/png/382109/1561455418605-fd99e403-1817-4103-aef3-046dfe619112.png#align=left&display=inline&height=308&originHeight=308&originWidth=715&size=0&status=done&width=715)

<a name="8haow"></a>
# 依赖注入的几种方式
通常，依赖注入可以通过**三种**方式完成，即：

- 接口注入
- 构造函数注入
- setter 注入
| 构造函数注入 | setter 注入 |
| --- | --- |
| 没有部分注入 | 有部分注入 |
| 不会覆盖 setter 属性 | 会覆盖 setter 属性 |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
| 适用于设置很多属性 | 适用于设置少量属性 |

目前，在 Spring Framework 中，仅使用构造函数和 setter 注入这**两种**方式。

<a name="zqWeC"></a>
## setter注入示例
```java
Public class Car {
    private String brand;
    public void setBrand(String brand) {
          this.brand = brand;
     }
     public String getBrand() {
          return this.brand;
     }
}
```

```xml
<bean id=“car” class=“com.jike.***.Car” >
    <property name=“brand”>
         <value>奔驰</value>
     </property>
</bean>

```


<a name="wjnMa"></a>
## 构造函数注入
```java
public class Car{
    private String brand;
    private double price;
    public Car(String brand, double price) {
        this.brand = brand;
        this.price = price;
    }
    public String getBrand() {
        return brand;
    }
    public void setBrand(String brand) {
        this.brand = brand;
    }
    public double getPrice() {
        return price;
    }
 
    public void setPrice(double price) {
        this.price = price;
    }
}

```


```xml
<bean id="Car" class="cn.lovepi.chapter02.reflect.Car">
        <constructor-arg type="String">
            <value>红旗CA72</value>
        </constructor-arg>
        <constructor-arg type="double">
            <value>26666</value>
        </constructor-arg>
</bean>

```


<a name="zjOLs"></a>
# Spring中的Ioc容器

<a name="JaQJF"></a>
## BeanFactory
BeanFactory 在 `spring-beans` 项目提供。BeanFactory ，就像一个包含 Bean 集合的工厂类。它会在客户端要求时实例化 Bean 对象。<br />

<a name="fOHW7"></a>
## ApplicationContext
ApplicationContext 在 `spring-context` 项目提供。ApplicationContext 接口扩展了 BeanFactory 接口，它在 BeanFactory 基础上提供了一些额外的功能。内置如下功能：

  - MessageSource ：管理 message ，实现国际化等功能。
  - ApplicationEventPublisher ：事件发布。
  - ResourcePatternResolver ：多资源加载。
  - EnvironmentCapable ：系统 Environment（profile + Properties）相关。
  - Lifecycle ：管理生命周期。
  - Closable ：关闭，释放资源
  - InitializingBean：自定义初始化。
  - BeanNameAware：设置 beanName 的 Aware 接口。

另外，ApplicationContext 会自动初始化非懒加载的 Bean 对象们。

<a name="XmA4d"></a>
## 两者之间有什么区别
| BeanFactory | ApplicationContext |
| --- | --- |
| 它使用懒加载 | 它使用即时加载 |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象 |
| 不支持国际化 | 支持国际化 |
| 不支持基于依赖的注解 | 支持基于依赖的注解 |

BeanFactory 粗暴简单，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 **“低级容器”**。<br />ApplicationContext 可以称之为 **“高级容器”**。因为他比 BeanFactory 多了更多的功能。他继承了多个接口。因此具备了更多的功能。例如资源的获取，支持多种消息（例如 JSP tag 的支持），对 BeanFactory 多了工具级别的支持等待。所以你看他的名字，已经不是 BeanFactory 之类的工厂了，而是 “应用上下文”， 代表着整个大容器的所有功能。该接口定义了一个 refresh 方法，此方法是所有阅读 Spring 源码的人的最熟悉的方法，用于刷新整个容器，即重新加载/刷新所有的 bean。<br />

<a name="1ldE0"></a>
## 常见的BeanFactory
BeanFactory 最常用的是 XmlBeanFactory 。它可以根据 XML 文件中定义的内容，创建相应的 Bean。

<a name="2JGvK"></a>
## 常见的ApplicationContext
以下是三种较常见的 ApplicationContext 实现方式：

- 1、ClassPathXmlApplicationContext ：从 ClassPath 的 XML 配置文件中读取上下文，并生成上下文定义。应用程序上下文从程序环境变量中取得。示例代码如下：

```java
ApplicationContext context = new ClassPathXmlApplicationContext(“bean.xml”);
```


- 2、FileSystemXmlApplicationContext ：由文件系统中的XML配置文件读取上下文。示例代码如下：

```java
ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”);
```


- 3、XmlWebApplicationContext ：由 Web 应用的XML文件读取上下文。例如我们在 Spring MVC 使用的情况。
- <br />

当然，目前我们更多的是使用 Spring Boot 为主，所以使用的是第四种 ApplicationContext 容器，ConfigServletWebServerApplicationContext 。

<a name="HVNXv"></a>
# 简述一下Spring Ioc的实现机制
```java
interface Fruit {

     public abstract void eat();
     
}
class Apple implements Fruit {

    public void eat(){
        System.out.println("Apple");
    }
    
}
class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}

class Factory {

    public static Fruit getInstance(String className) {
        Fruit f = null;
        try {
            f = (Fruit) Class.forName(className).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
    
}

class Client {

    public static void main(String[] args) {
        Fruit f = Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f != null){
            f.eat();
        }
    }
    
}
```

- Fruit 接口，有 Apple 和 Orange 两个实现类。
- Factory 工厂，通过反射机制，创建 `className` 对应的 Fruit 对象。
- Client 通过 Factory 工厂，获得对应的 Fruit 对象。
- 😈 实际情况下，Spring IoC 比这个复杂很多很多，例如单例 Bean 对象，Bean 的属性注入，相互依赖的 Bean 的处理，以及等等。

