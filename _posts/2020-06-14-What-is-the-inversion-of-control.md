---
layout: post
title: 理解Spring系列——什么是控制反转（Inversion of Control, IoC）
date: 2020-06-14
catalog: true
categories: 理解Spring系列
tags: 
- Spring
- 控制反转
- IoC
- 理解Spring系列
---
> 声明：我已委托「维权骑士」（rightknights.com）为我的文章进行维权行动。

# 控制反转 —— 软件复用的解决方案。
控制反转（Inversion of Control, IoC）最早由Michael Mattsson在《Object-Oriented Frameworks：A survey of methodological issues》一文中提出。


Wikipedia对于IoC的定义如下：
> In software engineering, inversion of control (IoC) is a programming principle. **IoC inverts the flow of control as compared to traditional control flow**. In IoC, custom-written portions of a computer program receive the flow of control from a generic framework. A software architecture with this design inverts control as compared to traditional procedural programming: in traditional programming, the custom code that expresses the purpose of the program calls into reusable libraries to take care of generic tasks, but with inversion of control, it is the framework that calls into the custom, or task-specific, code.


# 控制反转导致反转了什么？？？


在Martin Flow的[《Inversion of Control Containers and the Dependency Injection pattern》](https://martinfowler.com/articles/injection.html#InversionOfControl)中提到，在早期的终端程序时代，用户界面主要是由程序本身控制，你的程序里有很多命令，当程序提醒用户输入名字后并在收到用户输入的名字后，会做出相关的响应处理；而在图形界面时代，UI Framework通常会包括一个主循环（Main Loop），你的编码工作只需要提供事件处理器（Event Handler），来处理对应事件下的业务逻辑即可，不需要关系事件如何参数、何时到来。如果将上述逻辑写成代码如下：

### 举例

**在终端模式的业务代码**
```java
public class Terminal {
    public static void main(String[] args) {
        while (true) {
            Command command = obtainCommonForInput();
            if (isInputNameCommand(command)) {
                // 编写收到姓名的业务逻辑
                handleNameInputed();
            } else {
                // todo 添加其他的逻辑，需要添加command类型判断并编写对应回调逻辑
            }
        }
    }
}
```

**在GUI模式的业务代码**
```java
public class YourBizListener extends OnButtonClickListener {
    public void onButtonClicked() {
        // 编写按钮按下的业务逻辑
    }
}
```
![](/imgs/the-program-dependency-for-terminal.png)

```java
public class GUI {
    public static void main(String[] args) {
        Application application = createApplication();
        registerListener(application, new YourBizListener());
        // 如果需要响应其他事件，只需要实现对应的事件处理接口，并注册
    }
}
```
![](/imgs/the-program-dependency-for-GUI.png)

### 设计分析


从**程序控制流依赖和源码依赖**的角度看，在Terminal中，程序控制流的方向和源码依赖方向保持相同，都是自上而下的；这意味着当你需要编写函数`main()`时，对应的函数必须已声明（源码依赖），这样你才能调用相关代码；编写main函数的过程，就是确定控制流依赖的过程，也是产生源码依赖的方向。总之，代码编写者确定程序的具体执行过程。但是，GUI中的情况有所改变，图中的源码依赖还是自上而下的，但是在事件响应的回调上，控制流的方向则自下而上的，和源码依赖方向相反。
站在Framework的角度上看，源码依赖的方式和控制流依赖的方向是相反的，这样使得Framework可以支持不同开发人员的使用场景。因此，我们也将IoC称之为好莱坞原则：不要给我们打电话，我们会给你打电话(don't call us, we'll call you)。

从**关注点分离（Separation of concerns, SoC）**角度看，IoC解决的问题是如何解耦when-to-do和what-to-do。
Framework通过抽象事件响应的执行过程，将事件响应从**具体函数的调用**转化**抽象接口`OnButtonClickListener`的调用**，解耦了**事件响应执行（when-to-do ）**和**事件响应实现（what-to-do）**的关系；在此基础上，开发人员实现接口`OnButtonClickListener`并向Framework注册，以达到响应相关按钮的点击事件的目的。从中可以看出，决定事件回调是否执行的关键代码并不由开发人员掌握，而由Framework决定，代码执行的决定权从开发人员手中转移到Framework中，和之前相比方向上出现了**反转**。

> 这里说的“代码执行的决定权”是指“代码执行的**直接**决定权”。<br/>
例如，Framework直接决定当用户点击某个按钮的时候，调用用户定义的哪个回调函数。但是，Framework还是可以通过对外提供配置修改接口让开发人员**间接**干预Framework的具体执行过程。例如，在GUI的例子中，如果用户禁用某个按钮，Framework将不会响应该按钮的点击事件。<br/>


从整体上看，IoC主要分为以下几步：

1. what-to-do协议约定（Framework）：针对具体的场景，约定开发人员和Framework的调用方式，通常协议会固化成一个可编程对象，例如特定路径下的文件、特定对象实例的某个属性、特定抽象类或者接口等；
2. when-to-do逻辑确定（Framework）：针对之前约定的可编程对象，Framework编程相关逻辑，就确定when to do；
3. what-to-do逻辑实现（开发人员）：针对之前约定的可编程对象，开发人员根据自身的业务，确定相关属性，实现相关逻辑；
4. what-to-do逻辑注册（开发人员或Framework）：通过Framework的注册接口将what-to-do和when-to-do关联起来。

> 关于what-to-do逻辑注册的动作，不一定是有开发人员完成的。例如，Spring Bean容器中基于包名ComponentScan的bean发现注册机制就是由开发人员和Spring Framework共同完成的；再例如Tomcat webapp的自动部署是由Tomcat自身实现的。



# IoC的实现方式
IoC的实现方式有很多，最为常见的可以分为以下几种：
- 依赖注入（Dependency Injection, DI）
- 模板方法模式 & 策略模式
- 服务定位（ServiceLoader）

##### 依赖注入（Dependency Injection, DI）

简单讲，DI描述的问题是客户端获取一个对象的时候，Framework自动将该对象所有依赖的对象进行关联，确保客户端获得到的对象可以直接使用，无需再继续额外配置。
在没有DI技术的方式下，用户每创建一个对象，都需要对其相关属性进行单独的配置。

通常，DI分为构造器注入、字段注入、接口注入等三种注入方式。他们的使用场景也会存在很大的不同：
- 构造器注入将依赖注入过程和对象实例化的过程捆绑在一起，可以确保对象在实例化后依赖已经注入完成，适合于对于依赖约束关系要求较强的使用场景，例如组件A强依赖于组件B，无组件B实例的情况下，组件A无法运行，这种场景建议使用构造器注入；
- 字段注入：字段注入中可以将依赖注入过程作为一个单独的实现，开发人员可以根据自身实现要求允许对象的相关依赖为null，影响其他的字段的主张；
- 接口注入往往需要相关Java类实现Framework提供的相关接口，和**Framework 环境**有较强的依赖关系。例如Spring Framework中的BeanFactoryAware等Aware接口的注入为服务组件提供BeanFactory、ApplicationContext、ResourceLoader、BeanName等环境感知能力。

##### 模板方法模式 & 策略模式
模板方法模式 & 策略模式是设计模式中IoC的典范。在实现上，他们都利用面向对象编程的多态特性，实现when to do和 what to do的关注点分离。
在模板方法模式中，抽象类决定 what-to-do协议约定，只关注when to do，对应具体类实现what to do，when to do 和what to do 的关系绑定是由类继承关系确定的；策略模式下，策略接口约定do的具体内容，Context环境类确定when to do，提供what to do 策略的管理，对应的策略实现类决定what to do的具体逻辑。

##### 服务定位（ServiceLoader）

服务定位（ServiceLoader）利用spi技术用于特定服务组件的加载，可以达到组件插件化的设计目的。

# 场景举例

- [Spring Bean容器](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans)
- Servlet 容器，如[Tomcat](http://tomcat.apache.org/)等
- [ServiceLoader](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html)
- [SpringFactoriesLoader](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/io/support/SpringFactoriesLoader.html)

# 参考资料

- [《Object-Oriented Frameworks：A survey of methodological issues》,Michael Mattsson](https://www.semanticscholar.org/paper/Object-Oriented-Frameworks-%3A-A-Survey-of-Issues-Mattsson/4492b8c6fdcbe2c5b1c24ab744b3ff12c30cc492)
- [《Inversion of Control Containers and the Dependency Injection pattern》](https://martinfowler.com/articles/injection.html#InversionOfControl)
