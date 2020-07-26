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


# 类  `BeanDefinition`  的继承体系

通过上一章对 `BeanFactory` 的深入解析，我们发现 `BeanFactory` 将Bean 对象的实例化过程抽象成  `BeanDefinition`  这个类进行表达。
相同的，通过分析  `BeanDefinition`  的类继承结构（如下图）。

![](/imgs/Spring-Framework-BeanDefinition-Hierarchical.png)

### 接口继承

从图中，我们发现 `BeanDefination` 主要继承于 `AttributeAccessor` 和 `BeanMetadataElement` 等两个接口。前者封装对对象属性的相关操作（例如添加、删除、修改）；后者则是描述 `BeanDefination` 的原始定义来源（the configuration source {@code Object} for this metadata element），例如如果是基于 Annotation 配置的 Spring Bean时， source 对应的可能是 `StandardMethodMetadata` （通过注解  `@Bean`  进行定义） 或者 `File` （通过  `@Component`  进行定义） 。

### 类实现

从图中，我们发现  `AttributeAccessorSupport`  提供通用的属性操作封装，  `BeanMetadataAttributeAccessor`  提供Source 相关读写接口；在  `AttributeAccessorSupport`  的基础上，将属性格式统一成类  `BeanMetadataAttribute`  ；最终，通过继承类  `BeanMetadataAttribute`  ，实现接口 `BeanDefinition`  ，我们得到抽象类 `AbstractBeanDefinition`  ， 这一基础的  `BeanDefination`  。

根据使用场景的不同， `AbstractBeanDefinition`  存在三个实现类：  `RootBeanDefinition`  （用于实例化bean对象，其他类型的  `BeanDefinition`  需要转换成   `RootBeanDefinition`  才能被 `BeanFactory` 进行实例化）、  `ChildBeanDefinition`  （通过parent 属性支持 `BeanDefinition`  数据的继承 ）、 `GenericBeanDefinition` （通用的 `BeanDefination` 定义）。 而通过  `@Bean`  、  `@Component`  等注解注册的  `BeanDefination` ， 一般都是  `AnnotatedGenericBeanDefinition`  或者  `ScannedGenericBeanDefinition` ， 两者的区别在前者是基于Class对象实例得到相关属性，而后者是基于ASM字节码解析得到的，**无须进行类加载**。

#  接口`BeanDefinition`  相关字段定义

|字段名（加粗的字段同时支持属性的写入读取，其他字段支持字段的读取）|含义|
|--|--|
|**parentName**| BeanDefination 的父节点，用于相关属性的继承 ，可以为空|
|**beanClassName**|bean的Class名称，可以为空|
|**scope**|bean的scope，IoC容器提供singleton和prototype两种scope，支持业务拓展，默认为singleton|
|**lazyInit**|是否支持懒加载：默认为false，即在IoC容器初始化是进行Bean的初始化，而不是等到应用获取bean时才进行初始化|
|**dependsOn**|描述当前bean初始化前依赖的Spring bean，BeanFactory 确保该初始化顺序|
|**autowireCandidate**|当用户未指明具体引用时，是否将当前bean作为其他bean的类型注入的候选对象集，默认为true|
|**primary**|结合上一个字段`autowireCandidate`，当该bean出现在候选对象集中，该bean是否为优先选择的对象，默认为false|
|**factoryBeanName**|用于实例化Bean的工厂类的Bean Name，可为空|
|**factoryMethodName**|用于实例化Bean的工厂类的方法名称，可为空|
|**constructorArgumentValues**|描述Bean初始化的相关函数|
|**propertyValues**|描述Bean 相关属性的值|
|**initMethodName**|描述Bean 内部的自定义的初始化方法|
|**destroyMethodName**|描述Bean 内部的自定义的销毁方法|
|**role**|Bean的角色，一般分为`APPLICATION`、`SUPPORT`、`INFRASTRUCTURE`|
|**description**|Bean描述|
|ResolvableType|Type对象在Spring中的抽象表达，提供较为友好的泛型类型提取|
|Abstract|是否为Abstract类型|
|ResourceDescription|对于BeanDefination定义来源的描述，用于日志等信息展示场景|


#  类`AbstractBeanDefinition` 的拓展字段

在接口 `BeanDefinition` 的基础上， `AbstractBeanDefinition` 还提供一下几个字段

- autowireMode： 描述的自动装配Bean的模式，分为	`AUTOWIRE_BY_NAME`（通过名称指示自动装配Bean属性，适用于所有Bean属性设置和构造器参数设置），`AUTOWIRE_BY_TYPE`（按类型自动装配Bean属性的常量（适用于所有Bean属性设置和构造器参数设置），	`AUTOWIRE_CONSTRUCTOR`（自动装配，适用于构造器参数设置），	`AUTOWIRE_NO`（不进行相关类型设置）。
- dependencyCheck：检测Bean的属性依赖，分为 `DEPENDENCY_CHECK_NONE` （不进行依赖检查）、 `DEPENDENCY_CHECK_OBJECTS` （仅检查对象引用）、 `DEPENDENCY_CHECK_SIMPLE` （仅检查简单类型）、 `DEPENDENCY_CHECK_ALL` (检测所有的依赖) 等
- qualifiers ：类型为 `Map<String, AutowireCandidateQualifier>`， 描述和类型相关的自动装配候选的限定符对象。
- methodOverrides：方法重写集，描述的Bean方法执行的重写信息，从实现上分为 `LookupOverride` 和 `ReplaceOverride`，对应 LookupMethod 和 ReplaceMethod ，具体应用场景可以参考[Method Injection](https://spring.io/blog/2004/08/06/method-injection/)。
- enforceInitMethod：是否强制开启自定义初始化方法，默认为false，逻辑参考`org.springframework.beans.factory.support.AbstractBeanDefinition#applyDefaults`
- enforceDestroyMethod：是否强制开启自定义销毁方法，默认为false，逻辑参考`org.springframework.beans.factory.support.AbstractBeanDefinition#applyDefaults`
- synthetic：当前类是否为合成的类


# BeanDefination 的检验

BeanDefination 中的上述字段存在一定的约束关系，具体逻辑在`org.springframework.beans.factory.support.AbstractBeanDefinition#validate`中，MethodOverrides 和  FactoryMethodName 不允许重复出现，由于两者初始化策略存在冲突。