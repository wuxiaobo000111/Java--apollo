# DefaultListableBeanFactory

>&nbsp;&nbsp;&nbsp;&nbsp;	XmlBeanFactory继承自DefaultListableBeanFactory,而DefaultListableBeanFactory是整个bean加载的核心部分,是Spring注册及加载bean的默认实现,而对于XmlBeanfactory与DefaultListableBeanFactory不同的地方其实是在XmlBeanFactory中使用了自定义的XML读取器,XmlBeanDefinitionReader,实现了个性化的BeanDefinitionReader读取, DefaultListableBeanFactory继承了AbstractAutowireCapableBeanFactory并实现了ConfigurableListableBeanFactory以及BeanDefinitionRegistry接口。以下是ConfigurableListableBeanFactory的继承关系。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/1.jpg?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp; 如下的列表中展现了类的作用。

| 类名| 作用|
|--|--|
|  |  |
|AliasRegistry|定义对alias的简单增删改等操作.|
|SimpleAliasRegistry|主要使用map作为alias的缓存,并对接口AliasRegistry进行实现。|
|SingletonBeanRegistry|定义对单例的注册及获取,|
|BeanFactory|定义获取bean及bean的各种属性。|
|DefaultSingletonBeanRegistry|对接口SingletonBeanRegistry各函数的实现.|
|HierarchicalBeanFactory|继承BeanFactory,也就是在BeanFactory定义的功能的基础上增加了对parentFactory的支持|
|BeanDefinitionRegistry|定义对BeanDefinition的各种增删改操作。|
|FactoryBeanRegistrySupport|在DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。|
|ConfigurableBeanFactory|提供配置Factory的各种方法。|
|ListableBeanFactory|根据各种条件获取bean的配置清单。|
|AbstractBeanFactory|综合FactoryBeanRegistrySupport和ConfigurableBeanFactory 的功能|
|AutowireCapableBeanFactory|提供创建bean、 自动注入、初始化以及应用bean的后处理器,|
|AbstractAutowireCapableBeanFactory|综合AbstractBeanFactory并对接口Autowire CapableBeanFactory进行实现.|
|ConfigurableListableBeanFactory|Beanfactory配置清单,指定忽略类型及接口等.DefaultListableBeanFactory:综合上面所有功能,主要是对Bean注册后的处理。|


>&nbsp;&nbsp;&nbsp;&nbsp;XmlBeanFactory使用XmlBeanDefinitionReader对配置文件进行解析,读取其中的bean。

# XmlBeanDefinitionReader

>&nbsp;&nbsp;&nbsp;&nbsp;	XML配置文件的读取是Spring中重要的功能,因为Spring的大部分功能都是以配置作为切入点的,那么我们可以从XmlBeanDefinitionReader中梳理一下资源文件读取、解析及注册的大致脉络,首先我们看看各个类的功能。

| 类名 | 类名 |
|--|--|
|  ResourceLoader|  定义资源加载器,主要应用于根据给定的资源文件地址返回对应的Resource.|  
|  BeanDefinitionReader|主要定义资源文件读取并转换为BeanDefinition的各个功能。|   
 |EnvironmentCapable|定义获取Environment方法,|
|DocumentLoader|定义从资源文件加载到转换为Document的功能,|
|AbstractBeanDefinitionReader|对EnvironmentCapable, BeanDefinitionReader类定义的功能进行实现。|
|BeanDefinitionDocumentReader|定义读取Docuemnt并注册BeanDefinition功能.|
|BeanDefinitionParserDelegate|定义解析Element的各种方法。|

>&nbsp;&nbsp;&nbsp;&nbsp;经过以上分析,我们可以梳理出整个XML配置文件读取的大致流程,如图2-6所示,在XmlBeanDifinitionReader中主要包含以下几步的处理:

```
        (1)通过继承自AbstractBeanDefinitionReader中的方法,来使用ResourLoader将资源文件路径转换为对应的Resource文件。
	(2)通过DocumentLoader对Resource文件进行转换,将Resource文件转换为Docurment文件。
	(3)通过实现接口BeanDefinitionDocumentReader的DefaultBeanDefinitionDocumentReader类对Document进行解析,
    并使用BeanDefinitionParserDelegate对Element进行解析。
```
