---
layout: post
title: 理解Spring系列——BeanFactory，Spring IoC的核心担当
date: 2020-07-14
catalog: true
tags: 
- Spring
- 源码分析
- BeanFactory
---
> 声明：我已委托「维权骑士」（rightknights.com）为我的文章进行维权行动。

# 导言

通过上一章[《理解Spring系列——什么是控制反转（Inversion of Control, IoC）》](/2020/06/14/What-is-the-inversion-of-control/)，我们了解到IoC编程模式的本质，通过协议约定来分离when-to-do和what-to-do两个关注点，实现模块间的解耦。

而在Spring Framework中，BeanFactory绝对是Spring IoC的核心担当：`BeanFactory` 在Spring中的主要作用，通过抽象Bean实例化的具体过程（体现在BeanDefinition定义），借助依赖注入（Dependency Inject，DI）的能力，实现基于元数据的业务对象自动装配，可以理解成增强版的[Guice](https://github.com/google/guice)，如下图所示。

![](/imgs/spring-ioc-container-magic.png)

# Spring中的BeanFactory体系

在Spring Framework中，`BeanFactory`作为非常基础的接口存在，提供了Bean注册能力（包括Bean查询、类型判断、scope判断等基础能力）；在继承体系上，可以分成`BeanFactory实现类`和`ApplicationContext`两条继承线路。

两者的关系如下：`BeanFactory实现类`属于外观模式的子系统（SubSystem）承担着外界提供Bean对象的主要责任，是功能实现的主体；`ApplicationContext`作为`BeanFactory实现类`的外观系统（Facade System），通过 `BeanFactory` 提供的接口结合具体的业务场景解析元数据（`BeanDefinition` ）的解析和注册，并进行Bean对象的初步加载。
另外，`ApplicationContext`继承于接口 `BeanFactory` ，并在`BeanFactory`的基础上，提供AoP能力的集成、消息资源的管理、事件发布机制、以及基于特定场景（Web系统、文件系统）的能力封装等主要功能。


`BeanFactory` 完整的继承关系如下图所示：
![](/imgs/Spring-Framework-BeanFactory-Hierarchical.png)

### BeanFactory 的主要实现类

从图中可以看出，`BeanFactory` 的主要实现类包括`StaticListableBeanFactory`、`DefaultListableBeanFactory`、`SimpleJndiBeanFactory`等三个公共类，分别面向**静态场景**、**默认枚举配置场景**和**JNDI**等场景，对应的实现思路也不尽相同。
- `StaticListableBeanFactory`可以看做是基于`Map<String，Object>`的简单封装，**不包括Bean的实例化过程**，需要用户手动注册后才可以使用，不具备依赖注入能力，是`BeanFactory`最为简单的实现；
- `DefaultListableBeanFactory`是Spring `BeanFactory`的默认实现，是Spring IoC容器中最为复杂的部分，在后续的章节我们会进行分析；
- `SimpleJndiBeanFactory`则是基于JNDI场景发现对应Bean。


### 类 DefaultListableBeanFactory 

类 DefaultListableBeanFactory 的继承关系如下图所示， 从中我们可以基本上了解到 类 DefaultListableBeanFactory 的基本功能。

![](/imgs/Spring-Framework-DefaultListableBeanFactory-Hierarchical.png)

从接口实现上看，类 DefaultListableBeanFactory 主要实现了以下基础的接口：
- **`AliasRegistry`** ：属于spring-core模块（别名能力不限制于Bean Name场景，因此定义在core模块中），提供String类型别名相关能力，对应的实现类为 SimpleAliasRegistry ，可以看做是简易版的 Map<String,String> 抽象能力，提供别名的增删改查相关能力；
- **`SingletonBeanRegistry`** ：属于spring-beans模块，提供单例bean对象的注册和查询能力，可以看做是简易版的 Map<String,Object> ；
- **`ListableBeanFactory：`**：属于spring-beans模块，在 BeanFactory 的基础上提供更进一步的非精确化查询能力，支持基于类型（ `Class` 和 `ResolvableType` ）的查询能力，返回的结果支持beanName 和`Map<BeanName，Bean>`；
- **`HierarchicalBeanFactory`**： 属于spring-beans模块，提供多层次的BeanFactory结构，允许多个BeanFactory之间存在继承关系。例如在 WebApplicationCpontext 的使用场景中， WebApplicationContext 依赖于基于XML配置的 XMLApplicationContext等；
- **`AutowireCapableBeanFactory`**：属于spring-beans模块，提供基于bean的自动装配能力，包括依赖查找（Dependence Lookup）和依赖注入（Dependence Inject）能力；
- **`ConfigurableBeanFactory`**： 属于spring-beans模块，提供 DefaultListableBeanFactory 相关数据的读取和配置接口，同时提供销毁bean 的能力。

另外，接口`BeanDefinitionRegistry` 提供了基于beanName的 BeanDefinition 的注册、删除和查询能力。

- ConfigurableListableBeanFactory
- ：



### ApplicationContext接口

从 ApplicationContext 的定义上，我们可以看出，接口 ApplicationContext 默认具备的能力