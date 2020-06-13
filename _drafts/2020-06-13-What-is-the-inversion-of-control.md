---
layout: post
title: 理解Spring系列——什么是控制反转（Inversion of Control, IoC）
date: 2020-06-13
catalog: true
tags: 
- Spring
- 控制反转
- 源码分析
- IoC
---
# 控制反转 —— 软件复用的解决方案。
控制反转（Inversion of Control, IoC）最早由Michael Mattsson在《Object-Oriented Frameworks：A survey of methodological issues》一文中提出。~~该文章详细的描述了基于面向对象开发的Framework的基本特征。~~




Wikipedia对于IoC的定义如下：
> In software engineering, inversion of control (IoC) is a programming principle. **IoC inverts the flow of control as compared to traditional control flow**. In IoC, custom-written portions of a computer program receive the flow of control from a generic framework. A software architecture with this design inverts control as compared to traditional procedural programming: in traditional programming, the custom code that expresses the purpose of the program calls into reusable libraries to take care of generic tasks, but with inversion of control, it is the framework that calls into the custom, or task-specific, code.

从上面文字中，我们可以发现IoC最主要的作用是相比传统的编程模式，IoC下的控制流会产生变化。
在Martin Flow的[《Inversion of Control Containers and the Dependency Injection pattern
》](https://martinfowler.com/articles/injection.html#InversionOfControl)中提到，在早期的终端程序时代，用户界面主要是由程序本身控制，你的程序里有很多命令，当程序提醒用户输入名字后并在收到用户输入的名字后，会做出相关的响应处理；而在图形界面时代，UI Framework通常会包括一个主循环（Main Loop），你的编码工作只需要提供事件处理器（Event Handler），来处理对应事件下的业务逻辑即可，不需要关系事件如何参数、何时到来。如果将上述逻辑写成代码如下：

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

public class GUI {
    public static void main(String[] args) {
        Application application = createApplication();
        registerListener(application, new YourBizListener());
        // 如果需要响应其他事件，只需要实现对应的事件处理接口，并注册
    }
}
```
上述两端代码的程序控制流关系和源码依赖关系如下：

![](/imgs/the-program-dependency-for-terminal.png)

![](/imgs/the-program-dependency-for-GUI.png)
在Terminal中，程序控制流的方向和源码依赖方向保持相同。这意味着当你需要编写函数main()时，对应的函数必须已声明（源码依赖），否则函数main()的逻辑（控制流依赖）无法编写展开；但是在GUI中，情况就有所改变。


// 此方法非必须，框架可以实现事件监听器自动发现、注册的能力，对外直接启动




从



> 在这里补充下，Framework和Class Library的区别：在软件开发的过程中，我们通过抽象部分组件（development-for-reuse）已达到软件复用(development-for-reuse)的目的，而第三方类库（Class Library）和 Framework都可以解决上述问题。
虽然，从开发者的角度看来，都是调用一段第三方代码，但是两者在控制流(Control of Flow)存在很大的不同：使用第三方类库的情况下，程序需要自行决定程序的控制流，但是在Framework的运行环境下，Framework可以完全掌管程序执行过程的控制流，非静态执行代码。



# 控制反转导致反转了什么——What aspect of control are they really inverting？

Robert C.Martin在《架构整洁之道》一书中的“面向对象编程”一节中


# IoC的实现方式
IoC和DI、DIP、IBP的关系。
实现
模板方法模式：Template method design pattern
策略模式：Strategy design pattern
服务定位：Service locator pattern
依赖注入：Dependency injection
Constructor injection
Setter injection
Interface injection
# 场景举例
Servlet 容器

# 控制反转和容器的关系
# 参考资料
