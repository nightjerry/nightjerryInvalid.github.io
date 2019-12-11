---
layout: post
title:  "注解"
date:   2019-12-11 14:34:00 +0800
categories: jekyll update
tags: java
---

[TOC]

注解通过`@interface`关键字进行定义

```java
public @interface TestAnnotation{
  //定义了名为TestAnnotation的注解
}
```

### 元注解
元注解是基本注解，是注解的注解，**用到自定义的注解上**
元注解有5种:
`@Retention`,`@Documented`,`@Target`,`@Inherited`,`@Repeatable`

#### @Retention
是保留期的意思;当应用到其他注解上的时候，说明这个注解的存活时间.
取值如下:  
* `RetentionPolicy.SOURCE` 注解只有在源码阶段保留，在编译器进行编译时将被丢弃忽视
* `RetentionPolicy.CLASS` 注解只被保留到编译进行的时候，不会被加载到JVM中
* `RetentionPolicy.RUNTIME` 注解可以保留到程序运行的时期，会被加载到JVM中，程序运行时可以获取到它们  

#### @Documented
此注解与文档有关，作用是将注解中的元素包含到javadoc中  
#### @Target
指定注解使用的场景，如类上，方法上，成员变量，方法参数等
取值如下:
- `ElementType.ANNOTATION_TYPE` 给一个注解进行注解
- `ElementType.CONSTRUCTOR` 用在构造方法上
- `ElementType.FIELD` 给属性进行注解
- `ElementType.LOCAL_VARIABLE` 给局部变量进行注解
- `ElementType.METHOD` 用在函数上
- `ElementType.PACKAGE` 对包进行注解
- `ElementType.PARAMETER` 对方法中的参数注解
- `ElementType.TYPE` 给类型注解，(类，接口，枚举)

#### @Inherited
继承, 用在类上，如果一个超类被@Inherited注解过的注解修饰的话，那么超类的子类没有被任何注解应用的话，子类就继承了超类的注解
```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test { }

@Test
public class A {}
public class B extends A { }
```
注解Test被`@Inherited`修饰，类A被Test注解，类B继承A，类B也拥有Test这个注解

#### @Repeatable
自java1.8引入，被@Repeatable修饰的注解，在同一个类上修饰时，允许多次引用
```java
@interface PersonArr {
    Person[] value();
}
@Repeatable(Persons.class)
@interface Person{
    String role default "";
}
@Person(role="coder")
@Person(role="pm")
public class Man{
}
```
上例中，@Repeatable修饰了@Person，@Repeatable括号内的类相当于一个容器注解
`容器注解`：即存放其他注解的地方,本身也是注解
必须有一个value()属性,属性类型是被@Repeatable修饰过的注解**数组**

### 注解的属性
也叫成员变量，注解只有成员变量，没有方法.成员变量以"无参的方法"形式声明,方法名就是成员变量名，返回值就是该成员变量的类型。
属性可以用默认值，使用`default`关键字修饰
```java
@Target
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation{
    int id();
    String msg() default "";
    boolean isTest() default false;
}
```
#### 属性的使用
赋值的方式是在注解的括号内以`key=value`形式，多属性间用`,`隔开.
```java
@TestAnnotation(id=3, msg="hello annotation")
public class Test{
}
```
> 注解中定义属性时，类型必须是8种基本数据类型(int,float,double,short,long,boolean,byte,char)，及String,枚举，类(接口)，注解和它们的数组

### java预置的注解
 @Deprecated
用来标记过时的元素，如过时的方法，类，成员变量等。
 @Override
子类要复写父类中被 @Override 修饰的方法
@SuppressWarnings
阻止警告
@SafeVarargs
参数安全类型注解，阻止编译器产生uncheck警告
@FunctionalInterface
函数式接口注解，java1.8特性,`函数式接口`就是具有一个方法的接口; 函数式接口可以转换为Lambda表达式

### 注解的提取
通过反射获取注解
java.lang.reflect反射包下`AnnotatedElement`接口，通过该接口的方法利用反射技术读取注解的信息，如`Constructor`,`Field`,`Method`,`Package`,`Class` 都实现了AnnotatedElement接口
```
Class: 类的class对象定义
Constructor: 类的构造器定义
Field: 类的成员变量定义
Method: 类的方法定义
Package: 类的包定义
```
AnnotatedElement相关的api

| 方法名称 | 返回值 | 说明 |
| --- | --- | --- |
| getAnnotation(Class<T> annotationClass) | <T extends Annotation> T  | 该元素如果存在指定类型的注解，则返回，否则返回null |
| getAnnotations() | Annotation[] | 返回调用者上存在的所有注解，包括从父类继承的 |
| isAnnotationPresent(Class<?extends Annotation> annotationClass) | boolean | 如果指定类型的注解存在此元素上，则返回true，否则返回false |
| getDeclaredAnnotations() | Annotation[] | 返回直接存在于此元素上的所有注解(不包括父类继承的注解),没有则返回长度为0的数组 |

