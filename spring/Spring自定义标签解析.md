# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;在DefaultBeanDefinitionDocumentReader.parseBeanDefinitions()方法中调用了BeanDefinitionParserDelegate.parseCustomElement用来解析配置文件中的自定义标签。首先我们来介绍一下如何自定义标签。

# Spring中如何自定义标签

# 自定义标签的步骤


```text
    1.创建一个需要扩展的组件.
    2.定义一个XSD文件描述组件内容。
    3.创建一个文件,实现BeanDefinitionParser接口,用来解析XSD文件中的定义和组件定义。
    4.创建一个Handler文件,扩展自NamespaceHandlerSupport, 目的是将组件注册到Spring容器。
    5.编写Spring.handlers和Spring.schemas文件。

```