# Spring 事务实现方式

其中事务实现的几种方式转载于:[https://blog.csdn.net/mufeng633/article/details/88552245](https://blog.csdn.net/mufeng633/article/details/88552245)

<a name="tMezO"></a>
# 事务的四种特性
事务具备ACID四种特性，ACID是Atomic（原子性）、Consistency（一致性）、Isolation（隔离性）和Durability（持久性）的英文缩写。<br />（1）原子性（Atomicity）<br />事务最基本的操作单元，要么全部成功，要么全部失败，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样。<br />（2）一致性（Consistency）<br />事务的一致性指的是在一个事务执行之前和执行之后数据库都必须处于一致性状态。如果事务成功地完成，那么系统中所有变化将正确地应用，系统处于有效状态。如果在事务中出现错误，那么系统中的所有变化将自动地回滚，系统返回到原始状态。<br />（3）隔离性（Isolation）<br />指的是在并发环境中，当不同的事务同时操纵相同的数据时，每个事务都有各自的完整数据空间。由并发事务所做的修改必须与任何其他并发事务所做的修改隔离。事务查看数据更新时，数据所处的状态要么是另一事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看到中间状态的数据。<br />（4）持久性（Durability）<br />指的是只要事务成功结束，它对数据库所做的更新就必须永久保存下来。即使发生系统崩溃，重新启动数据库系统后，数据库还能恢复到事务成功结束时的状态。


<a name="MYHMS"></a>
# 事务的传播特性
事务传播行为就是多个事务方法调用时，如何定义方法间事务的传播。Spring定义了7中传播行为：<br />（1）propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，          这是Spring默认的选择。<br />（2）propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。<br />（3）propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。<br />（4）propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。<br />（5）propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。<br />（6）propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。<br />（7）propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与                            propagation_required类似的操作。

<a name="Ovdnl"></a>
# 事务的隔离级别

（1）read uncommited：是最低的事务隔离级别，它允许另外一个事务可以看到这个事务未提交的数据。<br />（2）read commited：保证一个事物提交后才能被另外一个事务读取。另外一个事务不能读取该事物未提交的数据。<br />（3）repeatable read：这种事务隔离级别可以防止脏读，不可重复读。但是可能会出现幻象读。它除了保证一个事务不能被另外一个事务读取未提交的数据之外还避免了以下情况产生（不可重复读）。<br />（4）serializable：这是花费最高代价但最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读之外，还避免了幻象读

<a name="18zu7"></a>
# 幻读、脏读、不可重复读
a.脏读：指当一个事务正字访问数据，并且对数据进行了修改，而这种数据还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。因为这个数据还没有提交那么另外一个事务读取到的这个数据我们称之为脏数据。依据脏数据所做的操作肯能是不正确的。

b.不可重复读：指在一个事务内，多次读同一数据。在这个事务还没有执行结束，另外一个事务也访问该同一数据，那么在第一个事务中的两次读取数据之间，由于第二个事务的修改第一个事务两次读到的数据可能是不一样的，这样就发生了在一个事物内两次连续读到的数据是不一样的，这种情况被称为是不可重复读。

c.幻读：一个事务先后读取一个范围的记录，但两次读取的纪录数不同，我们称之为幻象读（两次执行同一条 select 语句会出现不同的结果，第二次读会增加一数据行，并没有说这两次执行是在同一个事务中）

<a name="BDxJr"></a>
# 事务实现的几种方式

（1）编程式事务管理对基于 POJO 的应用来说是唯一选择。我们需要在代码中调用beginTransaction()、                         commit()、rollback()等事务管理相关的方法，这就是编程式事务管理。<br />（2）基于 TransactionProxyFactoryBean的声明式事务管理<br />（3）基于 @Transactional 的声明式事务管理<br />（4）基于Aspectj AOP配置事务

<a name="oxVMr"></a>
## 使用 TransactionTemplate 事务模板对象
首先：因为我们使用的是特定的平台，所以，我们需要创建一个合适我们的平台事务管理PlateformTransactionManager。如果使用的是JDBC的话，就用DataSourceTransactionManager。注意需要传入一个DataSource，这样，平台才知道如何和数据库打交道。<br />
<br />第二： 为了使得平台事务管理器对我们来说是透明的，就需要使用TransactionTemplate。使用TransactionTemplat需要传入一个PlateformTransactionManager 进入，这样，我们就得到了一个TransactionTemplate，而不用关心到底使用的是什么平台了。<br />
<br />第三： TransactionTemplate 的重要方法就是 execute 方法，此方法就是调用TransactionCallback 进行处理。实际上我们需要处理的事情全部都是在 TransactionCallback 中编码的。<br />
<br />第四： 也就是 TransactionCallback 接口，我们可以定义一个类并实现此接口，然后作为TransactionTemplate.execute 的参数。把需要完成的事情放到 doInTransaction中，并且传入一个TransactionStatus 参数。此参数是来调用回滚的。 也就是说 ，PlateformTransactionManager 和 TransactionTemplate 只需在程序中定义一次，而TransactionCallback 和 TransactionStatus 就要针对不同的任务多次定义了。<br />

<a name="CFwt5"></a>
### 实现例子
1.配置事务管理器

```xml
<!-- 配置事务管理器 ,封装了所有的事务操作,依赖于连接池 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	  <property name="dataSource" ref="dataSource"></property>
</bean>

```
2.配置事务模板对象

```xml
<!-- 配置事务模板对象 -->
 <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
     <property name="transactionManager" ref="transactionManager"></property>
  </bean>

```
3.Test

```java
@Controller
@RequestMapping("/tx")
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class TransactionController {

	@Resource
	public TransactionTemplate transactionTemplate;

	@Resource
	public PlatformTransactionManager transactionManager;
	
	@Resource
	public DataSource dataSource;

	private static JdbcTemplate jdbcTemplate;

	private static final String INSERT_SQL = "insert into cc(id) values(?)";
	private static final String COUNT_SQL = "select count(*) from cc";

	@Test
	public void TransactionTemplateTest(){
		//获取jdbc核心类对象,进而操作数据库
		jdbcTemplate = new JdbcTemplate(dataSource);
		//通过注解 获取xml中配置的 事务模板对象
		transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
		//重写execute方法实现事务管理
		transactionTemplate.execute(new TransactionCallbackWithoutResult() {
			@Override
			protected void doInTransactionWithoutResult(TransactionStatus status) {
				jdbcTemplate.update(INSERT_SQL, "33");   //字段sd为int型，所以插入肯定失败报异常，自动回滚，代表TransactionTemplate自动管理事务
			}
		});
		int i = jdbcTemplate.queryForInt(COUNT_SQL);
		System.out.println("表中记录总数："+i);
	}

}

```

<a name="biJKK"></a>
## 使用事务管理器 PlatformTransactionManager 对象
此方式，可手动开启、提交、回滚事务。

<a name="sB0BO"></a>
### 实现例子
1.配置事务管理

```xml
<!-- 配置事务管理 ,封装了所有的事务操作,依赖于连接池 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	  <property name="dataSource" ref="dataSource"></property>
</bean>

```
2.test

```java
@Controller
@RequestMapping("/tx")
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class TransactionController {

   @Resource
	public PlatformTransactionManager transactionManager;
	
	@Resource
	public DataSource dataSource;
	
	private static JdbcTemplate jdbcTemplate;

	private static final String INSERT_SQL = "insert into cc(id) values(?)";
	private static final String COUNT_SQL = "select count(*) from cc";

	@Test
	public void showTransaction(){
		//定义使用隔离级别，传播行为
		DefaultTransactionDefinition def = new DefaultTransactionDefinition();
		def.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
		def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
		//事务状态类，通过PlatformTransactionManager的getTransaction方法根据事务定义获取；获取事务状态后，Spring根据传播行为来决定如何开启事务
		TransactionStatus transaction = transactionManager.getTransaction(def);
		jdbcTemplate = new JdbcTemplate(dataSource);
		int i = jdbcTemplate.queryForInt(COUNT_SQL);
		System.out.println("表中记录总数："+i);
		try {
			jdbcTemplate.update(INSERT_SQL,"2");
			jdbcTemplate.update(INSERT_SQL,"是否");//出现异常，因为字段为int类型，会报异常，自动回滚
			transactionManager.commit(transaction);
		}catch (Exception e){
			e.printStackTrace();
			transactionManager.rollback(transaction);
		}
		int i1 = jdbcTemplate.queryForInt(COUNT_SQL);
		System.out.println("表中记录总数："+i1);
	}
}

```

<a name="rxsrj"></a>
## 基于Aspectj AOP开启事务
1.配置事务通知

```xml
<!-- 	   配置事务增强 -->
<tx:advice id="txAdvice"  transaction-manager="transactionManager">
    <tx:attributes>
	      <tx:method name="*" propagation="REQUIRED" rollback-for="Exception" />
    </tx:attributes>
</tx:advice>

```
2.配置织入

```xml
<!--aop代理事务。扫描 cn.sys.service 路径下所有的方法 -->
 <aop:config>
    <!--     扫描 cn.sys.service 路径下所有的方法，并加入事务处理 -->
    <aop:pointcut id="tx"  expression="execution(* cn.sys.service.*.*(..))" />
     <aop:advisor advice-ref="txAdvice" pointcut-ref="tx" />
 </aop:config>

```

<a name="ULdDv"></a>
## 基于注解的 @Transactional 的声明式事务管理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.2.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx-3.2.xsd">
           
       <!-- 创建加载外部Properties文件对象 -->
       <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
       		<property name="location" value="classpath:dataBase.properties"></property>
       </bean>
    <!-- 引入redis属性配置文件 -->
    <import resource="classpath:redis-context.xml"/>

       <!-- 配置数据库连接资源 -->
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" scope="singleton">
       		<property name="driverClassName" value="${driver}"></property>
       		<property name="url" value="${url}"></property>
       		<property name="username" value="${username}"></property>
       		<property name="password" value="${password}"></property>

       		<property name="maxActive" value="${maxActive}"></property>
       		<property name="maxIdle" value="${maxIdle}"></property>
       		<property name="minIdle" value="${minIdle}"></property>
       		<property name="initialSize" value="${initialSize}"></property>
       		<property name="maxWait" value="${maxWait}"></property>
       		<property name="removeAbandonedTimeout" value="${removeAbandonedTimeout}"></property>
       		<property name="removeAbandoned" value="${removeAbandoned}"></property>

       		<!-- 配置sql心跳包 -->
       		<property name= "testWhileIdle" value="true"/>
			<property name= "testOnBorrow" value="false"/>
			<property name= "testOnReturn" value="false"/>
			<property name= "validationQuery" value="select 1"/>
			<property name= "timeBetweenEvictionRunsMillis" value="60000"/>
			<property name= "numTestsPerEvictionRun" value="${maxActive}"/>
       </bean>

       <!--创建SQLSessionFactory对象  -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
       		<property name="dataSource" ref="dataSource"></property>
       		<property name="configLocation" value="classpath:MyBatis_config.xml"></property>
       </bean>

       <!-- 创建MapperScannerConfigurer对象 -->
       <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
       		<property name="basePackage" value="cn.sys.dao"></property>
       </bean>

       <!-- 配置扫描器   IOC 注解 -->
	   <context:component-scan base-package="cn.sys" />

	   <!-- 配置事务管理 ,封装了所有的事务操作,依赖于连接池 -->
	   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	   		<property name="dataSource" ref="dataSource"></property>
	   </bean>

        <!-- 配置事务模板对象 -->
       <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
            <property name="transactionManager" ref="transactionManager"></property>
        </bean>

<!-- 	  配置事务增强 -->
	   <tx:advice id="txAdvice"  transaction-manager="transactionManager">
      	<tx:attributes>
	      	<tx:method name="*" propagation="REQUIRED" rollback-for="Exception" />
      	</tx:attributes>
       </tx:advice>
       
<!--     aop代理事务 -->
       <aop:config>
      	<aop:pointcut id="tx"  expression="execution(* cn.sys.service.*.*(..))" />
      	<aop:advisor advice-ref="txAdvice" pointcut-ref="tx" />
      </aop:config>
</beans>

```

```java
	@Transactional
    public int saveRwHist(List<RwHist> list) {
       return rwDao.saveRwHist(list);
    }

```

