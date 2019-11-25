---
layout: post
title:  "java 反射"
date:   2019-11-25 16:30:02 +0800
categories: jekyll update
tags: java
---

## 反射Reflection
[TOC]

通过反射，在运行时获取程序每一个类型的成员和成员的信息
一般的对象类型在编译期就确定下来的，反射机制可以动态的创建对象并调用其属性； 
反射的核心: JVM在运行时才动态加载类或调用方法/访问属性，不需要事先知道运行对象是谁
简单说，反射可以加载运行时才得知名称的.class文件，获悉完整结构，并生成对象实体，或对fields变量设值，或调用methods

### 获取Class对象
* `Class.forName(String className)`
* 直接获取一个对象的class 

 ```java
    Class<?> clazz = int.class;
    Class<?> clazzInt = Integer.TYPE;    
```
* 调用对象的`getClass()`方法:
* ```java
    StringBuilder str = new StringBuilder("123");
    Class<?> clazz = str.getClass();
```

### 创建实例
通过反射来生成对象有两种方式：

1. 使用Class对象的`newInstance`方法来创建Class对象对应类的实例

    ```java
    Class<?> clazz = String.class;
    Object str = c.newInstance();    
    ```
2. 通过Class对象获取指定的Constructor对象，再调用`Constructor.newInstance()`方法来创建实例；这种方法可以用指定的构造器构造类的实例
    
    ```java
    Class<?> clazz = String.class;
    Constructor con = clazz.getConstructor(String.class);
    Object obj = con.newInstance("123");
    ```
### 获取方法
获取Class对象的方法集合，有3种方式

* `getDeclaredMethods`方法返回类或接口声明的所有方法，包括`public`,`protected`,`package`,`private`修饰的方法，但不包括继承的方法。
    
```java
public Method[] getDeclaredMethods() throws SecurityException
```

* `getMethods`方法返回某个类的所有`public`修饰的方法，包括继承的`public`方法

```java
public Method[] getMethods() throws SecurityException
```

* `getMethod`返回一个特定的方法(必须是`public`修饰的，不然会报错)，其中第一个参数为方法名称，后面的参数为方法的参数对应Class对象; `getDeclaredMethod`返回一个特定的方法;

```java
public Method getMethod(String name, Class<?> ... parameterTypes)
public Method getDeclaredMethod(String name,Class<?> ... parameterTypes)
```

```java
public class Test {
    public static void main(String[] args) {

        try {
            Class<TestRef> refClass = TestRef.class;
//            Object obj = refClass.newInstance();
            Method[] methods = refClass.getMethods();
            Method[] declaredMethods = refClass.getDeclaredMethods();

            Method add = refClass.getMethod("add", int.class, int.class);

            System.out.println("getMethods获取的方法：");
            for (Method method : methods) {
                System.out.println(method);
            }
            System.out.println("getDeclaredMethods获取的方法：");
            for (Method method : declaredMethods) {
                System.out.println(method);
            }

        }catch (NoSuchMethodException e){
            e.printStackTrace();
        }

    }

    class TestRef{
        public final int f = 3;

        public int add( int a, int b){
            return a+b;
        }

        @Override
        public String toString() {
            return super.toString();
        }
    }
}

```
执行结果

```java
getMethods获取的方法：
public int com.jerry.algorithm.mode.Test$TestRef.add(int,int)
public java.lang.String com.jerry.algorithm.mode.Test$TestRef.toString()
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()

getDeclaredMethods获取的方法：
public int com.jerry.algorithm.mode.Test$TestRef.add(int,int)
public java.lang.String com.jerry.algorithm.mode.Test$TestRef.toString()
```

### 获取类的成员变量
* `getField`： 访问公有的成员变量
* `getDeclaredField`: 所有已声明的成员变量，但不能得到其父类的成员变量

### 调用方法
通过`invoke`来调用方法

```java
public Object invoke(Object obj, Object ... args)
```
```java
Class<TestRef> refClass = TestRef.class;
Method method = refClass.getMethod("add", int.class, int.class);
Object obj = refClass.newInstance();
//私有方法设置此属性，否则报错
//method.setAccessible(true);
Object result = method.invoke(obj, 1, 3);
System.out.println(result); //4
```
### 访问私有方法
使用`setAccessible(true)`,获取访问权；如果不设置，会报错
`java.lang.IllegalAccessException: Class MainClass can not access a member of class `

### 修改私有常量
对于`int`,`long`,`boolean`,`String`这些基本数据类型JVM会优化，对于`Integer`,`Long`,`Boolean`,`Object`这种包装类型，JVM不会优化;[详细见此](https://juejin.im/post/598ea9116fb9a03c335a99a4)

对于基本类型的静态变量，JVM在编译期间会把引用此常量的代码替换成具体的常量值；

> 在程序运行时刻依然可以使用反射修改常量的值；但如果有函数引用常量值，JVM编译期间会替换为常量值，则修改常量的值不会替换到方法中；

```java
class Demo{
    private final String VALUE = "abc";

    public String getValue(){
        return VALUE;
    }
}
```
```java
public static void main(String[] args){
        Class<?> clazz = Demo.class;
        try{
            Object obj = clazz.newInstance();

            Method method = clazz.getDeclaredMethod("getValue");
            method.setAccessible(true);
            
            Field field = clazz.getDeclaredField("VALUE");
            if (field != null){
                field.setAccessible(true);
                System.out.println("Before modify: VALUE="+field.get(obj));
            }
            
            field.set(obj, "ABC");
            System.out.println("after modify: VALUE="+field.get(obj));
            System.out.println("method:getValue="+method.invoke(obj));
            
        }catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (java.lang.InstantiationException |java.lang.reflect.InvocationTargetException e){
            e.printStackTrace();
        }catch (java.lang.IllegalAccessException | java.lang.NoSuchMethodException e){
            e.printStackTrace();
        }
}
//执行结果：
Before modify: VALUE=abc
after modify: VALUE=ABC
method:getValue=abc
```


