---
layout: post
title:  "设计模式_创建型模式"
date:   2019-11-25 16:39:02 +0800
categories: jekyll update
tags: java design_pattern
---

#创建型模式
[TOC]
## 工厂模式 Factory Pattern
相当于创建实例对象的`new`;
可能多做一些工作，但有更大的可扩展性和尽量少的修改量;
创建对象时，不会对调用者暴露创建逻辑，通过使用通用接口来指向新创建的对象;

使用场景：不同条件下创建不同实例时，即子类较多;

**核心思想**:子类实现工厂接口，由工具类提供创建对象的方法，返回接口实例；

优点：扩展性高，屏蔽实现，调用方便；

缺点：没增加一个产品时，都需要增加一个具体类和对象实现工厂，增加了系统复杂度，和对工具类的依赖；

> 复杂对象适合用工厂模式，简单对象使用`new`创建
> 任何需要生成复杂对象的地方，都可以使用工厂模式

```java
//1.定义接口
public interface Animal{
    void eat();
}
//2.创建接口实体类(有共同性)
public class Cat implements Animal{
    @Override
    public void eat(){
        System.out.println("cat eat fish");
    }
}
public class Panda implements Animal{
    @Override
    public void eat(){
        System.out.println("panda eat bamboo");
    }
}
//3.创建工厂类，用于创建对象
public class AnimalFactory{
    public Animal create(String type){
        if ("cat".equals(type)){
            return new Cat();
        }else if("panda".equals(type)){
            return new Panda();
        }
        return null;
    }
}
//4.使用工厂模式
public class Demo{
    public static void main(String[] args){
        AnimalFactory factory = new AnimalFactory();
        Animal cat = factory.create("cat");
        cat.eat();
        Animal panda = factory.create("panda");
        panda.eat();
    }
}
//cat eat fish
//panda eat bamboo
```


## 单例模式 Signleton Pattern
保证java应用程序中，一个类Class只有一个实例存在

优点：

* 节省内存，因为限制了实例的个数
* 有利于Java垃圾回收

```java
//1.懒汉式 [lazy初始化，多线程安全]
/*
优点：第一次调用才初始化，避免内存浪费。
缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率
*/
public class Singleton{
    private static Singleton instance;
    private Singleton(){}
    //加载时初始化
    public static synchronized Singleton getInstance(){
        if (instance == null) {
            instance = new Singleton()
        }
        return instance;
    }
}
```
```java
//2.饿汉式 [lazy初始化：否； 多线程安全]
/*
优点：没有加锁，执行效率会提高。
缺点：类加载时就初始化，浪费内存。
*/
public class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
        return instance;  
    }  
}
``` 
```java
//3.双重校验锁(DCL，即 double-checked locking)
[lazy初始化； 多线程安全]
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
        if (singleton == null) {  
            synchronized (Singleton.class) {  
                if (singleton == null) {  
                    singleton = new Singleton();  
                }     
            }  
        }  
        return singleton;  
    }  
}
```
```java
//4.静态内部类 [lazy初始化，多线程安全]
/*
能达到双检锁方式一样的功效，但实现更简单
这种方式是 Singleton 类被装载了，instance 不一定被初始化。因为 SingletonHolder 类没有被主动使用，只有通过显式调用 getInstance 方法时，才会显式装载 SingletonHolder 类，从而实例化 instance。
*/
public class Singleton {  
    private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
    
    private Singleton (){}  
    public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE;  
    }  
}
```
```java
//5.枚举单例 [lazy初始化:否； 多线程安全]
/*
可读性低，不建议使用
*/
public enum Singleton {  
     INSTANCE;  
     public void doSomeThing() {  
     }  
 }  
```
### volatile关键字

## 建造者模式(Builder Pattern)
将一个复杂对象的构建与它的表示分离，使同样的构建过程可以创建不同的表示.
属于创建型模式，它提供了一种创建对象的最佳方式。
何时使用：

* 一些基本部件不会变，而其组合经常变化的时候。
* 复杂的对象,有很多大量组成部分,过程很复杂; Builder模式就是为了将部件与组装过程分开

优点：

* 建造者独立，易扩展
* 便于控制细节风险

缺点：

* 产品必须有共同点，范围有限制
* 如内部变化复杂，会有很多的建造类

> 注意事项：与工厂模式的区别是：建造者模式更加关注与零件装配的顺序

结合retrofit源码中的使用场景，进行举例

retrofit中`Retrofit`和`HttpServiceMethod`对象的创建就是通过建造者模式；
实现方式： 在类中创建静态内部类Builder，Builder提供外部访问权限，定义相关变量的set方法，并定义`build`函数最终创建外部类对象. Builder中函数遵循链式调用规则

```java
//Retrofit.java
public static final class Builder {
    public Builder() {...}
    
    public Builder callFactory(okhttp3.Call.Factory factory) {
      this.callFactory = checkNotNull(factory, "factory == null");
      return this;
    }
    public Builder callbackExecutor(Executor executor) {
      this.callbackExecutor = checkNotNull(executor, "executor == null");
      return this;
    }
    ...
    //创建外部类对象
    public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }
      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }
      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }
      List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>(this.callAdapterFactories);
      callAdapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));
      List<Converter.Factory> converterFactories =
          new ArrayList<>(1 + this.converterFactories.size());
      converterFactories.add(new BuiltInConverters());
converterFactories.addAll(this.converterFactories);
//给各变量赋值，并以此为参数创建Retrofit对象
      return new Retrofit(callFactory, baseUrl, unmodifiableList(converterFactories),
          unmodifiableList(callAdapterFactories), callbackExecutor, validateEagerly);
    }
}

//Retrofit构造函数可见性为`default`,本包内可见
Retrofit(okhttp3.Call.Factory callFactory, HttpUrl baseUrl,
      List<Converter.Factory> converterFactories, List<CallAdapter.Factory> callAdapterFactories,
      @Nullable Executor callbackExecutor, boolean validateEagerly) {
    this.callFactory = callFactory;
    this.baseUrl = baseUrl;
    this.converterFactories = converterFactories; // Copy+unmodifiable at call site.
    this.callAdapterFactories = callAdapterFactories; // Copy+unmodifiable at call site.
    this.callbackExecutor = callbackExecutor;
    this.validateEagerly = validateEagerly;
}
```


## 原型模式 Prototype Pattern
用于创建重复的对象，同时又能保证性能

当直接创建对象的代价比较大时，则采用这种模式
`implements Cloneable，重写 clone()`
优点： 
1、性能提高。 
2、逃避构造函数的约束。
缺点:
1，配备克隆方法需要对类的功能进行通盘考虑,特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候
2、必须实现 Cloneable 接口。

浅拷贝实现 Cloneable，重写clone，深拷贝是通过实现 Serializable 读取二进制流。

```java
public class Shape implements Cloneable {
    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }

    public Object clone() {
        Object clone = null;
        try {
            clone = super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }
}

public class Test {
    public static void main(String[] args) {
        Shape rect = new Shape();
        rect.setId("12");
        Shape rect1 = (Shape) rect.clone();
        System.out.println(rect1.getId());
    }
}
```


