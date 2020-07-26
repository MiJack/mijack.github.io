---
layout: post
title: 理解Spring系列——前言
date: 2020-06-07
catalog: true
categories: 理解Spring系列
tags: 
- Spring
- 理解Spring系列
---


> 声明：我已委托「维权骑士」（rightknights.com）为我的文章进行维权行动。

# 为什么会有《理解Spring》系列？ 

在日常开发工作中，笔者天天与Spring打交道，发现Spring系列工程的代码设计愈发有趣，因此专门开辟一个系列博客用于Spring代码设计相关的技术积淀和交流。


在Spring Framework官方介绍的[Design Philosophy](https://docs.spring.io/spring/docs/current/spring-framework-reference/overview.html#overview-philosophy)一节中，Spring 代码大都遵循着如下两点：
- 面向开放的设计：例如通过外部化配置动态的调整程序行为，通过扩展点实现自定义流程；
- 关注代码质量：Spring 大多数API在历经多次major版本后，基本与原始的版本保持一致；同时，提供较为健全的Java Doc，无环的代码依赖关系。

以Spring AOP为例，AOP的大部分代码都是基于Spring 在IoC模块预留的扩展点实现的；从代码依赖角度，Spring AOP模块依赖于Spring IoC模块，先有IoC再有AOP，可见IoC模块设计的拓展性；在此基础上，才有了后续的Spring Transaction功能。再例如，Spring同时支持基于XML配置、 Annotation 配置以及两者结合的BeanDefinition注册。其中主要原因，Spring Framework将 BeanDefinition 注册状态的维护在Spring Beans模块，将 BeanDefinition 的解析（从XML或者Configuration Class中解析）与注册行为在Spring Context模块。当然，Spring Boot中的自动装配机制也是基于Spring Framework完成的。上述几个例子，都在说明Spring 代码面向开放的设计思想。

但是，仅仅从Spring Framework的源码上看，就有21个gradle module，在算上其他工程模块，Spring 模块更是多如牛毛。好在每个模块有自己的特有功能，他们之间的依赖关系也有迹可循，在一定程度上可以帮助我们理解spring 的运作机理。从全局上看，Spring 是Bean容器（Core Container），提供Bean的管理，支持不同类型的Context实现（例如基于XML配置与Annotation配置、Application场景与Web 场景）；在Bean容器的基础上，通过面向切面编程（AOP）技术与消息机制进行更多场景的支持，例如数据访问、Web服务等。

![Spring Framework概览](https://docs.spring.io/spring/docs/4.0.x/spring-framework-reference/html/images/spring-overview.png)


# 内容大纲
源码分析主要分为以下几个部分：

- Spring IoC篇（spring-beans、spring-context等模块）
    - [什么是控制反转](/2020/06/14/What-is-the-inversion-of-control/)
    - [BeanFactory，Spring IoC的核心担当](/2020/07/14/BeanFactory-in-Spring/)
    - [BeanDefinition，Bean的实例化抽象表达](/2020/07/26/BeanDefinition-in-Spring-Framework/)
    - Spring Bean 的生命周期
    - Spring Bean 的依赖管理
    - Spring Bean 的Scope管理
    - Spring IoC 容器的拓展点
    - BeanDefinition的注册
    - BeanFactory体系及拓展
- Spring AOP篇（spring-aop模块）
    - AOP 基础概念
    - Spring 对 AOP 的基础封装
    - Spring AOP和 Spring IoC的关系
- Spring MVC篇（spring-web、spring-webmvc、spring-webflux等模块）
    - Spring 对WebServer的理解
    - Spring 对MVC的
    - Spring MVC的拓展点
    - Spring WebFlux介绍
- Spring 数据访问篇（spring-jdbc、spring-tx、spring-orm等模块）
    - Spring 事务
    - Spring Dao
- Spring 番外篇
    - Spring Framework 代码概览
    - Spring Integration（Spring Framework）
    - Spring Security
    - Spring Shell：基于Spring开发的命令行工具
    - Spring代码中的奇巧淫技

> 注：伴随着系列博客的更新，内容会与上述列表有所不同，以最终博客更新为准，如果相关章节完成，文字会链接到对应的博客地址。

# 说明

本系列涉及的相关工程的代码版本、文档链接如下：

|工程|版本|commit版本号|
|---|---|--||--
|[spring-framework](https://github.com/spring-projects/spring-framework)|[5.2.6.RELEASE](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/)|[b6d61062735b372a9daa8fa9078bef59bc4d8973](https://github.com/spring-projects/spring-framework/commit/b6d61062735b372a9daa8fa9078bef59bc4d8973)|
|[spring-security](https://github.com/spring-projects/spring-security)|[5.3.2.RELEASE](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/)|[532e546355ee1adb22a6ec3ee05d04cc698d2b06](https://github.com/spring-projects/spring-security/commit/532e546355ee1adb22a6ec3ee05d04cc698d2b06)|
|[spring-integration](https://github.com/spring-projects/spring-integration)|[5.3.0.RELEASE](https://docs.spring.io/spring-integration/docs/5.3.0.RELEASE/reference/html/)|[8fd0f068543f64253f1e495e7b1c8b17f2506194](https://github.com/spring-projects/spring-integration/commit/8fd0f068543f64253f1e495e7b1c8b17f2506194)|
|[spring-shell](https://github.com/spring-projects/spring-shell)|[2.0.0.RELEASE](https://docs.spring.io/spring-shell/docs/2.0.0.RELEASE/reference/htmlsingle)|[a6ecde14068cb300037b9decd5a30955d19c7011](https://github.com/spring-projects/spring-shell/commit/a6ecde14068cb300037b9decd5a30955d19c7011)|
