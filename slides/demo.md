---
layout: keynote
title: 理解Spring系列——BeanFactory，Spring IoC的核心担当
date: 2020-07-14
catalog: true
tags: 
- Spring
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

## `DefaultListableBeanFactory`的接口继承关系

从**接口实现**上看，类 DefaultListableBeanFactory 主要实现了以下基础的接口：
- **`AliasRegistry`** ：属于spring-core模块（别名能力不限制于Bean Name场景，因此定义在core模块中），提供String类型别名相关能力，对应的实现类为 SimpleAliasRegistry ，可以看做是简易版的 Map<String,String> 抽象能力，提供别名的增删改查相关能力；
- **`SingletonBeanRegistry`** ：属于spring-beans模块，提供单例bean对象的注册和查询能力，可以看做是简易版的 Map<String,Object> ；
- **`ListableBeanFactory：`**：属于spring-beans模块，在 BeanFactory 的基础上提供更进一步的非精确化查询能力，支持基于类型（ `Class` 和 `ResolvableType` ）的查询能力，返回的结果支持beanName 和`Map<BeanName，Bean>`；
- **`HierarchicalBeanFactory`**： 属于spring-beans模块，提供多层次的BeanFactory结构，允许多个BeanFactory之间存在继承关系。例如在 WebApplicationCpontext 的使用场景中， WebApplicationContext 依赖于基于XML配置的 XMLApplicationContext等；
- **`AutowireCapableBeanFactory`**：属于spring-beans模块，提供基于bean的**自动装配**能力，包括依赖查找（Dependence Lookup）和依赖注入（Dependence Inject）能力；
- **`ConfigurableBeanFactory`**： 属于spring-beans模块，提供 DefaultListableBeanFactory 相关数据的读取和配置接口，同时提供销毁bean 的能力。

- **`ConfigurableListableBeanFactory`** 在 `AutowireCapableBeanFactory` 提供了部分依赖的设置能力，例如，注册特定类型的依赖默认值（ `registerResolvableDependency(Class<?> dependencyType, Object autowiredValue)` ），判断某个bean依赖关系是否存在候选集合（`boolean isAutowireCandidate(String beanName, DependencyDescriptor descriptor)`），依赖忽略特定的接口或者类型的依赖注入（`ignoreDependencyType(Class<?> type)` 、`void ignoreDependencyInterface(Class<?> ifc)`）。依赖候选集的查询需要容器提供基于类型查询对应的bean的能力，因此接口`ConfigurableListableBeanFactory`继承了接口`ListableBeanFactory`，因此，在实现的所有接口中， `ConfigurationListableBeanFactory` 的继承关系是最为复杂的，聚合了很多抽象的能力，**是最接近实现类 `DefaultListableBeanFactory` 的接口**，因此在spring 在对外提供BeanFactory预处理接口`BeanFactoryPostProcessor`的时候，对应的方法签名是 `void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) `，而不是~~`void postProcessBeanFactory(BeanFactory beanFactory) `~~，这样也可以避免`BeanFactoryPostProcessor#postProcessBeanFactory`中传入的参数是ApplicationContext。

- 接口`BeanDefinitionRegistry` 提供了基于beanName的 BeanDefinition 的注册、删除和查询能力。

## `DefaultListableBeanFactory`的类继承关系

`DefaultListableBeanFactory` 的类继承关系，自上到下依次对应的是 
Object <- SimpleAliasRegistry <- DefaultSingletonBeanRegistry <- FactoryBeanRegistrySupport <- AbstractBeanFactory  <- AbstractAutowireCapableBeanFactory  <- DefaultListableBeanFactory  。

从上到下，BeanFactory 关注点变得越来越具体，逻辑也变得越来越复杂，对应的能力逐渐强大：从最初的别名注册能力，到bean对象的注册管理和`FactoryBean` 对象的管理，再到BeanFactory配置管理，自动装配能力的实现，最后才是BeanDefination注册管理，支持根据BeanDefination进行对象的实例化。

### `BeanFactory` vs `FactoryBean`

1. 两者的区别在：前者是工厂类，简单理解成 beanName和bean对象映射关系的维护者（是个容器），提供根据beanName查询bean对象的能力；后者是工厂类，描述的是Bean对象实例化的过程，用于生成特定类型的对象。
BeanFactory is a factory, FactoryBean is a bean。
2. `FactoryBean` 当你向容器注册名字为 factoryBeanName 的 FactoryBean的时候，你向容器注册的是 名字为&factoryBeanName的FactoryBean的对象，，通过factoryBeanName获取到的是 `FactoryBean#getObject` 返回的对象，该对象不受Spring 容器管理，具体参考[What's a FactoryBean?](https://spring.io/blog/2011/08/09/what-s-a-factorybean)。
3. 当创建Bean的过程中涉及到多个依赖对象的复杂配置（不是简单的属性注册），或者存在一定的复用性时，可以通过 `FactoryBean` 简化一部分实例过程，减少无关Bean的注册。例如 `AbstractEntityManagerFactoryBean` 相关实现。

### 实例说明

```java

@Configuration
public class FactoryBeanDemo {
    public static final String beanName = "user";

    @Bean(beanName)
    public FactoryBean<User> factoryBean() {
        return new FactoryBean<User>() {
            public User getObject() {
                User user = new User();
                user.setName("mijack");
                return user;
            }

            public Class<?> getObjectType() {
                return User.class;
            }
        };
    }

    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(FactoryBeanDemo.class);
        System.out.println(applicationContext.getBean(beanName) instanceof User);// true
        System.out.println(applicationContext.getBean(beanName) instanceof FactoryBean); // false
        System.out.println(applicationContext.getBean(BeanFactory.FACTORY_BEAN_PREFIX + beanName) instanceof User);//false
        System.out.println(applicationContext.getBean(BeanFactory.FACTORY_BEAN_PREFIX + beanName) instanceof FactoryBean);//true
        System.out.println(BeanFactoryUtils.beanNamesForTypeIncludingAncestors(applicationContext, User.class));// [user]
    }
}

```


### ApplicationContext 的类家族

![](/imgs/Spring-Framework-ApplicationContext-Hierarchical.png)

从 `ApplicationContext` 继承的接口上看，接口 `ApplicationContext` 继承了接口 `ListableBeanFactory` 、 `HierarchicalBeanFactory` 、 `EnvironmentCapable` 、`MessageSource` 、 `ApplicationEventPublisher` 、 `ResourcePatternResolver`，提供IoC容器能力、环境管理、事件发布机制、事件发布-响应机制、资源定位加载、i18n的能力。


从 `ApplicationContext` 实现的子类上看，ApplicationContext根据不用应用场景分成不同的实现类：
- 应用在普通场景，还是web场景？
- 配置文件是从XML中读取还是从Config Class中读取，或者基于Groovy脚本配置？基于XML读取的方式资源路径是感觉文件系统来读取，还是基于Classpath来读取？

图中中蓝框部分通过  `ApplicationContext` ->  `ConfigurableApplicationContext` -> `AbstractApplicationContext` -> `AbstractRefreshableApplicationContext` -> `AbstractRefreshableConfigApplicationContext` -> `AbstractRefreshableWebApplicationContext` &  `AbstractXmlApplicationContext` 的继承体系实现了相关功能，但是这样存在一定问题：不同场景下，我文件加载的方式都要重新实现一遍，例如实现XML的配置加载，需要分成文件系统和Web应用两种不同的实现类，代码复用度太低。

为了避免上述情况，进一步提高代码的利用率，Spring 提出 `GenericApplicationContext`类继承体系（红色框的实现体系）： `GenericApplicationContext` 实现了接口 `BeanDefinitionRegistry`，将资源注册过程从核心流程中剥离出来，特定场景下数据加载解析，有子类实现，提供友好的API接口。不同于 `AbstractRefreshableApplicationContext` （维护内部的 `BeanFactory` ，支持刷新时重新创建 BeanFactory ）， `GenericApplicationContext` 中的 beanFactory 支持从外界导入，为了避免复杂的状态管理， `GenericApplicationContext` 默认不支持容器的重新刷新。