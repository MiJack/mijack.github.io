---
layout: post
title: 理解Spring系列——BeanDefinition，Bean的实例化抽象表达
date: 2020-07-26
catalog: true
categories: 理解Spring系列
tags: 
- Spring
- BeanDefinition
- 理解Spring系列
---
> 声明：我已委托「维权骑士」（rightknights.com）为我的文章进行维权行动。


# 类 BeanDefination 的继承体系

通过上一章对 BeanFactory 的深入解析，我们发现 BeanFactory 将Bean 对象的实例化过程抽象成 BeanDefination 这个类进行表达。
相同的，通过分析 BeanDefination 的类继承结构（如下图）。

![](/imgs/Spring-Framework-BeanDefinition-Hierarchical.png)

### 接口继承

从图中，我们发现 BeanDefination 主要继承于 AttributeAccessor 和 BeanMetadataElement 等两个接口。前者封装对对象属性的相关操作（例如添加、删除、修改）；后者则是描述 BeanDefination 的原始定义来源（the configuration source {@code Object} for this metadata element），例如如果是基于 Annotation 配置的 Spring Bean时， source 对应的可能是StandardMethodMetadata （通过注解 @Bean 进行定义） 或者 File （通过 @Component 进行定义） 。

### 类实现

从图中，我们发现 AttributeAccessorSupport 提供通用的属性操作封装， BeanMetadataAttributeAccessor 提供Source 相关读写接口；在 AttributeAccessorSupport 的基础上，将属性格式统一成类 BeanMetadataAttribute ；最终，通过继承类 BeanMetadataAttribute ，实现接口 BeanDefinition ，我们得到抽象类 AbstractBeanDefinition ， 这一基础的 BeanDefination。

根据使用场景的不同，AbstractBeanDefinition 存在三个实现类： RootBeanDefinition （用于实例化bean对象，其他类型的 BeanDefination 需要转换成  RootBeanDefinition 才能被 BeanFactory 进行实例化）、 ChildBeanDefinition （通过parent 属性支持BeanDefinition数据的继承 ）、GenericBeanDefinition （通用的BeanDefination定义）。 而通过 @Bean 、 @Component 等注解注册的 BeanDefination， 一般都是 AnnotatedGenericBeanDefinition 或者 ScannedGenericBeanDefinition， 两者的区别在前者是基于Class对象实例得到相关属性，而后者是基于ASM字节码解析得到的，无须进行类加载。

# BeanDefination 相关字段定义

|字段名|含义|备注|
|**parentName**|||
|**beanClassName**|||
|**scope**|||
|**lazyInit**|||
|**dependsOn**|||
|**autowireCandidate**|||
|**primary**|||
|**factoryBeanName**|||
|**factoryMethodName**|||
|**constructorArgumentValues**|||
|**PropertyValues**|||
|**initMethodName**|||
|**destroyMethodName**|||
|**role**|||
|**description**|||
|ResolvableType|||
|Singleton|||
|Prototype|||
|Abstract|||
|ResourceDescription|||
|OriginatingBeanDefinition|||

# BeanDefination 的初始化方式

#### 基于构造函数的初始化
#### 基于BeanClass 的静态工厂方法的初始化
#### 基于特定Bean的工厂方法的初始化


# Bean 的生命周期

## 初始化


- BeanNameAware's setBeanName
- BeanClassLoaderAware's setBeanClassLoader
- BeanFactoryAware's setBeanFactory
- EnvironmentAware's setEnvironment
- EmbeddedValueResolverAware's setEmbeddedValueResolver
- ResourceLoaderAware's setResourceLoader (only applicable when running in an application context)
- ApplicationEventPublisherAware's setApplicationEventPublisher (only applicable when running in an application context)
- MessageSourceAware's setMessageSource (only applicable when running in an application context)
- ApplicationContextAware's setApplicationContext (only applicable when running in an application context)
- ServletContextAware's setServletContext (only applicable when running in a web application context)
- postProcessBeforeInitialization methods of BeanPostProcessors
- InitializingBean's afterPropertiesSet
- a custom init-method definition
- postProcessAfterInitialization methods of BeanPostProcessors

## BeanFactory关闭


- postProcessBeforeDestruction methods of DestructionAwareBeanPostProcessors
- DisposableBean's destroy
- a custom destroy-method definition
