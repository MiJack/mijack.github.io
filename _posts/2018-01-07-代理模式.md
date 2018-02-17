---
layout: post
catalog: true
title: 代理模式
date: 2018-01-07 20:30:28
tags: 
- 设计模式
- 代理模式
- CGLib
---



> 声明：我已委托「维权骑士」（rightknights.com）为我的文章进行维权行动。


# 代理模式

代理模式是设计模式中一种常见的设计模式，我们往往通过代理模式可以拦截目标方法的执行，进行自己想要的业务需求，例如日志拦截，权限校验等工作。

代理模式的实现方式如下：

![代理模式类图](/imgs/代理模式类图.png)

在代理模式中，我们将类分为以下三类角色

抽象对象Subject，代理对象Proxy，以及真实对象RealSubject。

当我们想要执行某一个动作时，我们不是直接调用RealSubject的方法，而是通过Proxy间接调用RealSubject的方法。为了使Proxy和RealSubject对外界提供功能保持一致，我们定义了一个接口Subject，表示他们可以提供的业务的能力，Proxy和RealSubject均实现了这一个接口，不同的是RealSubject是真正的业务实现，而Proxy只是简单调用了字段subject对应的函数。

## 代理模式的好处

代理模式通过引入代理对象这一角色，在一定程度上隔离了业务发起方和业务实际处理方，避免了两者之间的直接交互，在不更改业务执行流程的前提下，为灵活的需求变化提供了可拓展的编码空间。

# 代理模式的简单实现——静态代理

对应的代码如下

```java
public interface Subject{
  void doAction1();
  void doAction2();
}
public class RealSubject implements Subject{
  
  public void doAction1(){
  	System.out.println("相关的业务事件 1");
  }
  public void doAction2(){
  	System.out.println("相关的业务事件 2");
  }
}
public class Proxy implements Subject{
  private Subject subject;
  public void doAction1(){
    if(subject!=null){
      subject.doAction1();
    }
  }
  
  public void doAction2(){
    if(subject!=null){
      subject.doAction2();
    }
  }
}

// 实际调用
// 创建realObject
Subject subject = new RealSubject();
// 创建代理对象 Proxy object
ProxySubject proxy= new ProxySubject();
// 绑定代理对象和real object
proxy.setSubject(subject);
// 调用Proxy的业务方法
proxy.doAction1();
```

上述代码就是传说中的静态代理模式。

## 静态代理的弊端

但是，这样的静态代理模式虽然实现起来简单，但是存在着很多弊端：

- 代理者使用了业务接口，因此需要**实现接口中声明的所有方法** ：当一个接口中定义了100个甚至更多的方法时，我们需要实现相应数量的方法。如果是好几个接口，那这个工作量可想而知；
- **业务逻辑的关注点分散在类中的各个方法中** ，增加了类功能维护的成本：我们如果想实现一个功能，但方法开始执行时，输出对应的日志信息。如果通过现有实现方式，代理对象的每个方法中我们都要调用对应的日志函数，工作量很大；另外，如果那一天，我们希望改下部分方法的处理逻辑，工作量依旧存在。



换而言之，静态代理的弊端就在于**代理对象没有对真实对象的动作行为做抽象处理，无形之中增加了程序逻辑修改带来的维护成本** 。

# JDK中的动态代理

为了解决上述问题，我们需要用到JDK中提供的动态代理API。具体API实现如下：

```java
Subject proxy = (Subject) Proxy.newProxyInstance(
        // 生成的字节码由该ClassLoader加载到虚拟器中
        ClassLoader.getSystemClassLoader(),
        // 需要代理的接口，是一个数组，可以代理多个对象
        new Class[]{Subject.class},
        new InvocationHandler() {
            // 代理对象的代理的真实对象
            Subject subject = new RealSubject();

          /**
           * @param proxy 系统生成的代理对象实例
           * @param method 客户端调用的目标方法的方法对象
           * @param args 客户端调用目标方法时传入的参数
           * @return
           * @throws Throwable
           */
          @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                // 调用前的逻辑
                // 调用代理对象的相关方法
                Object result = method.invoke(subject, args);
                // 调用后的逻辑
                // 返回处理结果
                return result;
            }
        });
// 调用代理对象的相关方法
proxy.doAction1();
```

## 动态代理的实现原理

JDK中的动态代理通过**字节码生成** 技术，产生**目标代理对象** ，将用户对目标接口的方法调用**统一路由到特定的接口** `java.lang.reflect.InvocationHandler` ，将客户端发起的调用转化成一一对应的Method对象，实现了代理动作的**抽象** ，实现代理的目的。

## 生成的代理类

通过设置相关的系统属性为true  ，我们可以在磁盘上看到`Proxy.newInstance()`方法生成的类的字节码文件。
```java
System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
```
例子中生成的代理类如下:

```java
package com.sun.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements Subject {
    private static Method m1;
    private static Method m4;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final void doAction1() throws  {
        try {
            super.h.invoke(this, m4, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void doAction2() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m4 = Class.forName("Subject").getMethod("doAction1");
            m3 = Class.forName("Subject").getMethod("doAction2");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}


```



生成代理类有以下特点：

- 该类继承于类java.lang.reflect.Proxy，实现了用户传入的interface数组中的所有接口，是一个final类；


- 如果代理的接口均为public的，那么代理类就是public的，对应的包名为com.sun.proxy;

- 如果代理的接口中存在非public接口，那么代理类就是非public的，对应的包名就是非public接口所在的包名;

- 方法签名重复：

  - 如果代理的接口中存在两个及以上的方法持有相同的方法签名，那么这些方法对应的返回类型也必须保持一致，不然对应的代理类型无法生成。
  - 对于方法签名重复的方法，代理类会根据传入的代理接口的顺序将指定目标方法对象为第一个声明该方法签名的接口。

- 代理对象还代理了常见的hashcode、equals、toString等方法。

  >注意事项:
  >
  >1. 也正是因为生成的代理类继承了类Proxy，Java不支持类的多继承，所以我们不能通过相同方式实现针对类的动态代理。
  >2. 方法签名相同是指方法名相同，同时对应位置参数类型也相同，不包含方法的返回类型



# 动态代理和静态代理的区别

## 是否生成新的代码

静态代理和动态代理最大的区别在于是否产生了新的类：

- 静态代理中所有的类均是由开发人员编写的；
- 动态代理中代理类是系统类Proxy的静态方法`newProxyInstance()`完成，用户无需关心代理类的具体实现，只需要关心如何处理**函数调用的逻辑** ；
- 方法逻辑的统一处理是动态代理的最大优点：
  - 两者比较发现，对于动态代理方法，代理类对于开发人员而言是透明的，他们不需要关心代理类中对接口的**底层实现** ，而只需要从方法抽象角度，通过同一个InvocationHandler接口决定每个方法在调用时的具体行为逻辑。


## 两者在时序图的差异

对比静态代理和动态代理的时序图中，我们可以发现静态代理的调用层数明显比动态代理少很多，而且不包含任何反射行为，在一定程度上执行效率要比动态调用高，但是Proxy和RealObject高度耦合到一起，这种实现方式对开放拓展不利。

![](/imgs/静态代理时序图.png)

但是，动态代理中的`Proxy`通过`InvocationHandler`对所有的函数调用请求做了统一处理，将所有请求转化成对应的`Method`对象，减少了`InvocationHandler`对`RealObject`的依赖，在一定程度上得到了解耦的目的，实现了

![](/imgs/动态代理时序图.png)



# 代理那些不能被JDK代理的类

前面我们已经提到由于Proxy生成的新的类继承了类Proxy，而Java不支持类的多继承，所以JDK的动态代理不支持针对类中方法的代理处理。但是，我们可以通过CGLib这个库实现这些类的代理。

例如，我们有一个类StudentService：

```java
public class Service {
    public void save(User user) {
        // do something
    }

  public void query(User user) {
        // do something
    }
}

```

我们想要代理这个类，在save执行前输出user对象，执行后输出存在完成的提示时，通过CGLib的具体实现代码如下：

```java
// 获取一个Enhancer对象，以生成新的字节码
Enhancer enhancer = new Enhancer();
// 设置目标类的父类
enhancer.setSuperclass(Service.class);
// 设置目标函数的回调函数Callback对象
enhancer.setCallback(new CglibProxy());
// 创建子类并生成对应的代理对象实例
Service service = (Service) enhancer.create();
// 调用目标函数
service.save(new User());
```

其中CGLibProxy的实现如下：

```java
public class CglibProxy implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, 
                            Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println(objects[0]);
        Object result = methodProxy.invokeSuper(o, objects);
        System.out.println("打印结束");
        return result;
    }
}
```

## CGlib中的那些callback

CGLib提供了很多Callback实现，MethodInterceptor只是其中的一个实现，其他的还有`NoOp`、`LazyLoader`、`Dispatcher` 、`InvocationHandler` 、`FixedValue`、`ProxyRefDispatcher`。他们的具体含义如下

- MethodInterceptor：全权代理所有方法的调用执行。
- NoOp：无额外逻辑，把方法调用直接委派到父类中的实现。
- LazyLoader：被代理的对象需要懒加载
- Dispatcher：将方法调用转发到特定的对象的方法上
- InvocationHandler：JDK中InvocationHandler在GCLib的实现，可以代理类中的方法
- FixedValue：对于特定方法，强制返回特定的值，常和CallbackFilter结合使用
- ProxyRefDispatcher：和Dispatcher作用相同，不同的是ProxyRefDispatcher可以找到代理对象。

## CallbackFilter

在上面例子里，我们采用了是MethodIntercepter，实际上，Enhancer支持多个Callback，我们可以通过CallbackFilter指派每一个方法的代理模式采用哪一种Callback。当然，这种指定关系是**生成的字节码**中方法的代理模式，是一种静态关系绑定，不是动态的。

具体代理实现如下：

```java
// 获取一个Enhancer对象，以生成新的字节码
Enhancer enhancer = new Enhancer();
// 设置目标类的父类
enhancer.setSuperclass(Service.class);
Callback[] callbacks = new Callback[]{new CglibProxy(), NoOp.INSTANCE};
// 设置目标函数的回调函数Callback对象数组
enhancer.setCallbacks(callbacks); // 指定代理类需要用到的所有的Callback，是一个数组
// 设置CallbackFilter
enhancer.setCallbackFilter(new CallbackFilter() { 
  @Override
  public int accept(Method method) {// 根据特定的方法返回对应的代理位置
    if (method.getName().equals("query")) {
      return 1;// => 对应的Callback数组的第1位
    }
    return 0;  // => 对应的Callback数组的第0位
  }
});
```


参考资料：

- [CGLib学习笔记](http://www.cnblogs.com/shijiaqi1066/p/3429691.html)