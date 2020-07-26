---
layout: post
title: 理解Spring系列—— Spring Bean 的生命周期
date: 2020-07-25
catalog: true
categories: 理解Spring系列
tags: 
- Spring
- 生命周期
- 理解Spring系列
---
> 声明：我已委托「维权骑士」（rightknights.com）为我的文章进行维权行动。


#  `BeanDefinition`  的初始化方式

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
