
# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;正如每一个java程序员会使用spring框架,但是对于Spring框架的底层源码真的认真研究过吗。这篇文章就作一个抛砖引玉的作用来简单介绍一下Spring如何实现将配置文件中的配置转化为bean的。首先我们先举个小例子,显式的调用一下。

# 代码

## pom

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.5.RELEASE</version>
        </dependency>
    </dependencies>
```

## 配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
    <bean class="com.bobo.spring.learn.base.MyTestBean" id="myTestBean"/>
</beans>
```

## 代码

```java
public class MyTestBean {

    private String testStr = "testStr";

    public void setTestStr(String testStr) {
        this.testStr = testStr;
    }

    public String getTestStr() {
        return testStr;
    }
}

 public static void main(String[] args) throws IOException {

        /**
         * 	super();
         * 		ignoreDependencyInterface(BeanNameAware.class);
         * 		ignoreDependencyInterface(BeanFactoryAware.class);
         * 		ignoreDependencyInterface(BeanClassLoaderAware.class);
         *  XmlBeanFactory构造函数中移除了这三个接口
         */
        XmlBeanFactory factory = new XmlBeanFactory(new ClassPathResource("mytest.xml"));
        MyTestBean bean = factory.getBean(MyTestBean.class);
        System.out.println(bean.getTestStr());
    }
```
>代码里是不是超级简单。但是不能想象的是Spring框架本身帮助我们做了很多的东西。下面就让我们探究一下Spring如何将配置文件中配置映射成为了Bean。


# ClassPathResource

>&nbsp;&nbsp;&nbsp;&nbsp;ClassPathResource实现了封装底层资源。
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/3.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;从上图我们可以看到,最顶层的是InputStreamSource。这个接口只有一个方法:InputStream getInputStream()。这个方法就是让底层资源转化为一个输入流。可以是File,Classpath下的资源、byte、array等等。Resource接口实现了对于底层资源的属性访问。

# XmlBeanFactory

>&nbsp;&nbsp;&nbsp;&nbsp;在XmlBeanFactory通过XmlBeanDefinitionReader去操作配置文件生成bean。代码如下所示：
```java
public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
		super(parentBeanFactory);
		this.reader.loadBeanDefinitions(resource);
	}


//通过传入ClassPathResource解析
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isTraceEnabled()) {
			logger.trace("Loading XML bean definitions from " + encodedResource);
		}

                //获取已经加载过的bean。
		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
		try {
                        //通过传入的参数生成一个输入流。
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
                                        //设置编码格式
					inputSource.setEncoding(encodedResource.getEncoding());
				}
                                        //然后去加载bean
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}

```

>&nbsp;&nbsp;&nbsp;&nbsp;doLoadBeanDefinitions是最为核心的一个方法。


```
                protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {

		try {
                    //通过两个参数,生成了Document对象。
			Document doc = doLoadDocument(inputSource, resource);
			int count = registerBeanDefinitions(doc, resource);
			if (logger.isDebugEnabled()) {
				logger.debug("Loaded " + count + " bean definitions from " + resource);
			}
			return count;
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}




    protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
		return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
				getValidationModeForResource(resource), isNamespaceAware());
	}

    public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		int countBefore = getRegistry().getBeanDefinitionCount();
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}

```

>&nbsp;&nbsp;&nbsp;&nbsp;doLoadBeanDefinitions这个方法做了三件事情。校验格式，生成Document对象,执行registerBeanDefinitions方法。