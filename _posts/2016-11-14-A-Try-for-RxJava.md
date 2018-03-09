---
layout: post
catalog: true
title: 小试RxJava原理
date: 2016-11-14
tags: 
- RxJava
- 源码分享

---

RxJava的Hello World
在开始介绍RxJava的源码之前，我们先来写一个RxJava的Hello world吧！

```java

Observable
        .create(new Observable.OnSubscribe<Integer>() {
            public void call(Subscriber<? super Integer> subscriber) {
                subscriber.onNext(1);
                subscriber.onNext(2);
                subscriber.onNext(3);
                subscriber.onCompleted();
            }
        })
        .subscribe(new Subscriber<Integer>() {
            public void onCompleted() {
                System.out.println("onCompleted");
            }
            public void onError(Throwable e) {
                System.out.println("Error");
            }
            public void onNext(Integer integer) {
                System.out.println(integer);
            }
        });
```

这是我们熟悉的RxJava链式调用，在源码分析上比较麻烦，无法很明确的声明每一个代码块的含义。在这里，我们采用非链式调用的方式对这个Hello world进行重构（我们采用lambda 表达式）。


```java
//创建OnSubscribe，在call()中对应数据源产生形式
Observable.OnSubscribe<Integer> onSubscribe = (subscriber) -> {
    subscriber.onNext(1);
    subscriber.onNext(2);
    subscriber.onNext(3);
    subscriber.onCompleted();
};
//创建Subscriber，定义针对数据流的响应
Subscriber<Integer> subscriber = new Subscriber<Integer>() {
    public void onCompleted() {
        System.out.println("onCompleted");
    }
    public void onError(Throwable e) {
        System.out.println("Error");
    }
    public void onNext(Integer integer) {
        System.out.println(integer);
    }
};
//根据OnSubscribe创建Observable
Observable<Integer> observable = Observable.create(onSubscribe);
//创建observable、subscriber的注册关系
observable.subscribe(subscriber);
```

# Hello World的简单分析

简单的说，RxJava的原理就是观察者模式，不过RxJava比观察者模式强大多了。在RxJava，Observable相当于被观察者，它是事件的源头，而OnSubscribe则是定义数据源如何发送事件，或者如何发送什么样的数据；Subscriber则是观察者（在代码实现上，Subscriber实现了接口Observer），定义了接收数据后对应的反应。observable.subscribe(subscriber)将两者进行了关联：即告诉Observable，它有一个Subscriber；同时触发OnSubscribe.onCall()，开启整个事件流。
如下图,明显问题的关键就在于observable.subscribe(subscriber)。



## observable.subscribe(subscriber)
接下来，我们对observable.subscribe(subscriber)进行分析,对应的代码在Observable.java中。

注：如果没有特殊说明，我们使用的RxJava的版本为1.2.2

```java

public final Subscription subscribe(Subscriber<? super T> subscriber) {
    return Observable.subscribe(subscriber, this);
}
static <T> Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable) {
    //检查subscriber、observable.onSubscribe是否为空
    if (subscriber == null) {
        throw new IllegalArgumentException("subscriber can not be null");
    }
    if (observable.onSubscribe == null) {
        throw new IllegalStateException("onSubscribe function can not be null.");
    }
    //调用subscriber的onStart()
    subscriber.onStart();
    //将subscriber封装成SafeSubscriber
    if (!(subscriber instanceof SafeSubscriber)) {
        subscriber = new SafeSubscriber<T>(subscriber);
    }
    try {
        //通过RxJavaHook对observable, observable.onSubscribe进行封装，同时调用Observable.OnSubscribe类的call(Subscriber)
        RxJavaHooks.onObservableStart(observable, observable.onSubscribe).call(subscriber);
        //通过RxJavaHook对subscriber进行封装，并返回结果
        return RxJavaHooks.onObservableReturn(subscriber);
    } catch (Throwable e) {
        //判断异常，必要时抛出
        Exceptions.throwIfFatal(e);
        //当Subscriber不再订阅时，有RxJavaHook负责处理
        if (subscriber.isUnsubscribed()) {
            RxJavaHooks.onError(RxJavaHooks.onObservableError(e));
        } else {
            // if an unhandled error occurs executing the onSubscribe we will propagate it
            try {
                subscriber.onError(RxJavaHooks.onObservableError(e));
            } catch (Throwable e2) {
                //判断异常，必要时抛出
                Exceptions.throwIfFatal(e2);
                //封装异常并抛出
                RuntimeException r = new OnErrorFailedException("Error occurred attempting to subscribe [" + e.getMessage() + "] and then again while trying to pass to onError.", e2);
                RxJavaHooks.onObservableError(r);
                throw r; 
            }
        }
        //取消订阅
        return Subscriptions.unsubscribed();
    }
}
```

核心的代码就是RxJavaHooks.onObservableStart(observable, observable.onSubscribe).call(subscriber)简化成observable.onSubscribe.call(subscriber)。这才是关键。这里的subscriber其实对应于我们编写的Subscriber。那么，很明显,Hello world的调用关系如下，看上去是不是很简单。

```java
observable.subscribe(subscriber);
-->onSubscribe.onCall(Subscriber)
    -->subscribe.onNext(1)       ==>  你写的subscriber.onNext(1)
    -->subscribe.onNext(2)       ==>  你写的subscriber.onNext(2)
    -->subscribe.onNext(3)       ==>  你写的subscriber.onNext(3)
    -->subscribe.onCompleted()   ==>  你写的subscriber.onCompleted()
```

说的再直白一点，为什么RxJava可以完成调用–响应呢？还记得你在Hello World里定义的Observable.OnSubscribe吗？看看它的onCall方法的参数的类型是不是和你定义的观察者Subscriber是同一个类型，都是rx.Subscriber。懂了吧！简单的理解，因为你在call(subscriber)中调用subscribe.onNext(1),所以你写的subscriber的onNext(Integer)方法会被调用。所以，你认为将你写的 subscriber变成了Observable.OnSubscribe里方法call(Subscriber)的一个参数.也只是因为你调用了observable.subscribe(subscriber),才有了后面onNext()、onCompleted()、onError()的一系列方法调用

# map
在解释完RxJava的Hello world，下面我们分析一下map，实例代码（非链式调用）如下：
```java
Observable.OnSubscribe<Integer> onSubscribe = (subscriber) -> {
    subscriber.onNext(1);
    subscriber.onNext(2);
    subscriber.onNext(3);
    subscriber.onCompleted();
};
Func1<Integer, String> func1 = (integer) -> {
    return "Integer " + integer;
};
Subscriber<String> subscriber = new Subscriber<String>() {...};
Observable<Integer> observable = Observable.create(onSubscribe);
//分析重点
Observable<String> mapObservable = observable.map(func1);
mapObservable.subscribe(subscriber);
```

要了解为什么map可以实现链式调用，那么我们需要深入到源码那一层进行分析。不过在此之前，我们可以想一下如果是我们进行开发，那么应该如何开发。这里提供一个实现思路：在这里，mapObservable其实承担着双重责任，它既是 observable对应的Subscriber(注册者)，也是subscriber的Observable(观察对象)。

```java
MapSubscriber mapSubscriber = new MapSubscriber(Func1);
Observable<Integer> observable = Observable.create(onSubscribe);
Observable<String> mapObservable = Observable.create((subscriber)->{/*doSomething*/});
mapObservable.subscribe(subscriber);
observable.subscribe(mapSubscriber);
```

查看源码，我们发现RxJava的具体实现如下：

Observable.java
```java

public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
    return create(new OnSubscribeMap<T, R>(this, func));
}
public static <T> Observable<T> create(OnSubscribe<T> f) {
    return new Observable<T>(RxJavaHooks.onCreate(f));
}
```

OnSubscribeMap.java
```java
public final class OnSubscribeMap<T, R> implements OnSubscribe<R> {
    final Observable<T> source;
    final Func1<? super T, ? extends R> transformer;
    public OnSubscribeMap(Observable<T> source, Func1<? super T, ? extends R> transformer) {
        this.source = source;
        this.transformer = transformer;
    }
    @Override
    public void call(final Subscriber<? super R> o) {
        MapSubscriber<T, R> parent = new MapSubscriber<T, R>(o, transformer);
        o.add(parent);
        source.unsafeSubscribe(parent);
    }
    static final class MapSubscriber<T, R> extends Subscriber<T> {
        final Subscriber<? super R> actual;
        final Func1<? super T, ? extends R> mapper;
        boolean done;
        public MapSubscriber(Subscriber<? super R> actual, Func1<? super T, ? extends R> mapper) {
            this.actual = actual;
            this.mapper = mapper;
        }
        @Override
        public void onNext(T t) {
            R result;
            try {
                result = mapper.call(t);
            } catch (Throwable ex) {
                Exceptions.throwIfFatal(ex);
                unsubscribe();
                onError(OnErrorThrowable.addValueAsLastCause(ex, t));
                return;
            }
            actual.onNext(result);
        }
        @Override
        public void onError(Throwable e) {
            if (done) {
                RxJavaHooks.onError(e);
                return;
            }
            done = true;
            actual.onError(e);
        }
        @Override
        public void onCompleted() {
            if (done) {
                return;
            }
            actual.onCompleted();
        }
        @Override
        public void setProducer(Producer p) {
            actual.setProducer(p);
        }
    }
}
```
