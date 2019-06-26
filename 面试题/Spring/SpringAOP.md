# Spring AOP

转载于：[http://www.tianxiaobo.com/2018/06/17/Spring-AOP-源码分析系列文章导读/#4总结](http://www.tianxiaobo.com/2018/06/17/Spring-AOP-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E5%AF%BC%E8%AF%BB/#4%E6%80%BB%E7%BB%93)

<a name="vSE1G"></a>
# AOP术语和相应的实现
AOP 全称是 Aspect Oriented Programming，即面向切面的编程，AOP 是一种开发理念。通过 AOP，我们可以把一些非业务逻辑的代码，比如安全检查，监控等代码从业务方法中抽取出来，以非侵入的方式与原方法进行协同。这样可以使原方法更专注于业务逻辑，代码结构会更加清晰，便于维护。

<a name="bz2CY"></a>
## 连接点-JoinPoint
连接点是指程序执行过程中的一些点，比如方法调用，异常处理等。在 Spring AOP 中，仅支持方法级别的连接点。上面是比较官方的说明，下面举个例子说明一下。现在我们有一个用户服务 UserService 接口，该接口定义如下：

```java
public interface UserService {
    void save(User user);
    void update(User user);
    void delete(String userId);
    User findOne(String userId);
    List<User> findAll();
    boolean exists(String userId);
}
```

![](https://cdn.nlark.com/yuque/0/2019/jpeg/382109/1561545127982-2f5ac69d-b709-486b-85e0-45325a8bb9f3.jpeg#align=left&display=inline&height=892&originHeight=892&originWidth=1592&size=0&status=done&width=1592)


如上所示，每个方法调用都是一个连接点。接下来，我们来看看连接点的定义：

```java
public interface Joinpoint {
    /** 用于执行拦截器链中的下一个拦截器逻辑 */
    Object proceed() throws Throwable;
    Object getThis();
    AccessibleObject getStaticPart();
}
```

这个 Joinpoint 接口中，proceed 方法是核心，该方法用于执行拦截器逻辑。关于拦截器这里简单说一下吧，以`前置通知拦截器`为例。在执行目标方法前，该拦截器首先会执行前置通知逻辑，如果拦截器链中还有其他的拦截器，则继续调用下一个拦截器逻辑。直到拦截器链中没有其他的拦截器后，再去调用目标方法。关于拦截器这里先说这么多，在后续文章中，我会进行更为详细的说明。<br />上面说到一个方法调用就是一个连接点，那下面我们不妨看一下`方法调用`这个接口的定义。如下：

```java
public interface Invocation extends Joinpoint {
    Object[] getArguments();
}

public interface MethodInvocation extends Invocation {
    Method getMethod();
}
```

如上所示，方法调用接口 MethodInvocation 继承自 Invocation，Invocation 接口又继承自 Joinpoint。看了上面的代码，我想大家现在对连接点应该有更多的一些认识了。接下面，我们来继续看一下 Joinpoint 接口的一个实现类 ReflectiveMethodInvocation。当然不是看源码，而是看它的继承体系图。如下：<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/382109/1561545276315-fc2c301b-b93f-475b-8129-cf9dbf36e93e.jpeg#align=left&display=inline&height=748&originHeight=748&originWidth=1598&size=0&status=done&width=1598)

关于连接点的相关知识，我们先了解到这里。有了这些连接点，接下来要做的事情是对我们感兴趣连接点进行一些横切操作。在操作之前，我们首先要把我们所感兴趣的连接点选中，怎么选中的呢？这就是切点 Pointcut 要做的事情了，继续往下看。

<a name="Mmk5A"></a>
## 切点-PointCut
刚刚说到切点是用于选择连接点的，那么应该怎么选呢？在回答这个问题前，我们不妨先去看看 Pointcut 接口的定义。如下：

```java
public interface Pointcut {

    /** 返回一个类型过滤器 */
    ClassFilter getClassFilter();

    /** 返回一个方法匹配器 */
    MethodMatcher getMethodMatcher();

    Pointcut TRUE = TruePointcut.INSTANCE;
}
```
Pointcut 接口中定义了两个接口，分别用于返回类型过滤器和方法匹配器。下面我们再来看一下类型过滤器和方法匹配器接口的定义：

```java
public interface ClassFilter {
    boolean matches(Class<?> clazz);
    ClassFilter TRUE = TrueClassFilter.INSTANCE;

}

public interface MethodMatcher {
    boolean matches(Method method, Class<?> targetClass);
    boolean matches(Method method, Class<?> targetClass, Object... args);
    boolean isRuntime();
    MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
}
```
上面的两个接口均定义了 matches 方法，用户只要实现了 matches 方法，即可对连接点进行选择。在日常使用中，大家通常是用 AspectJ 表达式对连接点进行选择。Spring 中提供了一个 AspectJ 表达式切点类 - AspectJExpressionPointcut，下面我们来看一下这个类的继承体系图：<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/382109/1561545376696-5bf9f4ee-63d7-443d-8b19-3cf31ad0bafe.jpeg#align=left&display=inline&height=542&originHeight=542&originWidth=1584&size=0&status=done&width=1584)

如上所示，这个类最终实现了 Pointcut、ClassFilter 和 MethodMatcher 接口，因此该类具备了通过 AspectJ 表达式对连接点进行选择的能力。那下面我们不妨写一个表达式对上一节的连接点进行选择，比如下面这个表达式：

```java
execution(* *.find*(..))
```

<br />该表达式用于选择以 find 的开头的方法，选择结果如下：<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/382109/1561545406478-d3ff0d33-fd1d-4bdc-8cc1-4f6558d8ac96.jpeg#align=left&display=inline&height=890&originHeight=890&originWidth=1596&size=0&status=done&width=1596)<br />
<br />通过上面的表达式，我们可以就可以选中 findOne 和 findAll 两个方法了。那选中方法之后呢？当然是要搞点事情。so，接下来`通知(Advice)`就该上场了。<br />

<a name="MEYk9"></a>
## 通知-advice
通知 Advice 即我们定义的横切逻辑，比如我们可以定义一个用于监控方法性能的通知，也可以定义一个安全检查的通知等。如果说切点解决了通知在哪里调用的问题，那么现在还需要考虑了一个问题，即通知在何时被调用？是在目标方法前被调用，还是在目标方法返回后被调用，还在两者兼备呢？Spring 帮我们解答了这个问题，Spring 中定义了以下几种通知类型：

- 前置通知（Before advice）- 在目标方便调用前执行通知
- 后置通知（After advice）- 在目标方法完成后执行通知
- 返回通知（After returning advice）- 在目标方法执行成功后，调用通知
- 异常通知（After throwing advice）- 在目标方法抛出异常后，执行通知
- 环绕通知（Around advice）- 在目标方法调用前后均可执行自定义逻辑

上面是对通知的一些介绍，下面我们来看一下通知的源码吧。如下：

```java
public interface Advice {

}
```
如上，通知接口里好像什么都没定义。不过别慌，我们再去到它的子类接口中一探究竟。

```java
/** BeforeAdvice */
public interface BeforeAdvice extends Advice {

}

public interface MethodBeforeAdvice extends BeforeAdvice {

    void before(Method method, Object[] args, Object target) throws Throwable;
}

/** AfterAdvice */
public interface AfterAdvice extends Advice {

}

public interface AfterReturningAdvice extends AfterAdvice {

    void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable;
}
```

从上面的代码中可以看出，Advice 接口的子类接口里还是定义了一些东西的。下面我们再来看看 Advice 接口的具体实现类 AspectJMethodBeforeAdvice 的继承体系图，如下：

![](https://cdn.nlark.com/yuque/0/2019/jpeg/382109/1561545509326-542408d4-9c17-4949-afa5-1c9c00787282.jpeg#align=left&display=inline&height=562&originHeight=562&originWidth=1592&size=0&status=done&width=1592)

现在我们有了切点 Pointcut 和通知 Advice，由于这两个模块目前还是分离的，我们需要把它们整合在一起。这样切点就可以为通知进行导航，然后由通知逻辑实施精确打击。那怎么整合两个模块呢？答案是，`切面`。好的，是时候来介绍切面 Aspect 这个概念了。

<a name="3TfaR"></a>
## 切面-Aspect
切面 Aspect 整合了切点和通知两个模块，切点解决了 where 问题，通知解决了 when 和 how 问题。切面把两者整合起来，就可以解决 对什么方法（where）在何时（when - 前置还是后置，或者环绕）执行什么样的横切逻辑（how）的三连发问题。在 AOP 中，切面只是一个概念，并没有一个具体的接口或类与此对应。不过 Spring 中倒是有一个接口的用途和切面很像，我们不妨了解一下，这个接口就是切点通知器 PointcutAdvisor。我们先来看看这个接口的定义，如下

```java
public interface Advisor {

    Advice getAdvice();
    boolean isPerInstance();
}

public interface PointcutAdvisor extends Advisor {

    Pointcut getPointcut();
}
```
简单来说一下 PointcutAdvisor 及其父接口 Advisor，Advisor 中有一个 getAdvice 方法，用于返回通知。PointcutAdvisor 在 Advisor 基础上，新增了 getPointcut 方法，用于返回切点对象。因此 PointcutAdvisor 的实现类即可以返回切点，也可以返回通知，所以说 PointcutAdvisor 和切面的功能相似。不过他们之间还是有一些差异的，比如看下面的配置：

```xml
<bean id="aopCode" class="xyz.coolblog.aop.AopCode"/>
    
<aop:config expose-proxy="true">
    <aop:aspect ref="aopCode">
    	<!-- pointcut -->
        <aop:pointcut id="helloPointcut" expression="execution(* xyz.coolblog.aop.*.hello*(..))" />

        <!-- advoce -->
        <aop:before method="before" pointcut-ref="helloPointcut"/>
        <aop:after method="after" pointcut-ref="helloPointcut"/>
    </aop:aspect>
</aop:config>
```
如上，一个切面中配置了一个切点和两个通知，两个通知均引用了同一个切点，即 pointcut-ref=“helloPointcut”。这里在一个切面中，一个切点对应多个通知，是一对多的关系（可以配置多个 pointcut，形成多对多的关系）。而在 PointcutAdvisor 的实现类中，切点和通知是一一对应的关系。上面的通知最终会被转换成两个 PointcutAdvisor，这里我把源码调试的结果贴在下面：<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/382109/1561545659980-7b85ac2b-7382-4bb4-90c2-ec961d738f12.jpeg#align=left&display=inline&height=400&originHeight=400&originWidth=1878&size=0&status=done&width=1878)

在本节的最后，我们再来看看 PointcutAdvisor 的实现类 AspectJPointcutAdvisor 的继承体系图。如下：<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/382109/1561545666884-7d38c1d1-11ad-47f9-9ff6-b35a1444e6e1.jpeg#align=left&display=inline&height=440&originHeight=440&originWidth=1584&size=0&status=done&width=1584)

<a name="KjSXD"></a>
# 织入-Weaving

现在我们有了连接点、切点、通知，以及切面等，可谓万事俱备，但是还差了一股东风。这股东风是什么呢？没错，就是织入。所谓织入就是在切点的引导下，将通知逻辑插入到方法调用上，使得我们的通知逻辑在方法调用时得以执行。说完织入的概念，现在来说说 Spring 是通过何种方式将通知织入到目标方法上的。先来说说以何种方式进行织入，这个方式就是通过实现后置处理器 BeanPostProcessor 接口。该接口是 Spring 提供的一个拓展接口，通过实现该接口，用户可在 bean 初始化前后做一些自定义操作。那 Spring 是在何时进行织入操作的呢？答案是在 bean 初始化完成后，即 bean 执行完初始化方法（init-method）。Spring通过切点对 bean 类中的方法进行匹配。若匹配成功，则会为该 bean 生成代理对象，并将代理对象返回给容器。容器向后置处理器输入 bean 对象，得到 bean 对象的代理，这样就完成了织入过程。
<a name="1Vib2"></a>
# 如何配置代理方式

1.基于注解方法：<br /><tx:annotation-driven transaction-manager="txManager" proxy-target-class="true"/><br />2.基于xml配置方法：<br /><aop:config expose-proxy="true" proxy-target-class="false"><br /></aop:config><br />默认false，选择jdbc代理模式，true使用cglib代理模式。


