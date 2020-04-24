---
layout: post
title:  "java 反射"
date:   2019-11-25 16:30:02 +0800
categories: jekyll update
tags: java
---

## 反射Reflection
[获取Class对象](#reflect_1)
[创建实例](#reflect_2)
[获取方法](#reflect_3)
[获取类的成员变量](#reflect_4)
[调用方法](#reflect_5)
[访问私有方法](#reflect_6)
[修改私有常量](#reflect_7)
[进阶](#reflect_8)
- [静态函数](#reflect_8_1)
- [反射接口 —— 动态代理](#reflect_8_2)
- [kotlin中函数类型作为参数的反射](#reflect_8_3)

[反射android自定义application](#reflect_9)

通过反射，在运行时获取程序每一个类型的成员和成员的信息
一般的对象类型在编译期就确定下来的，反射机制可以动态的创建对象并调用其属性； 
反射的核心: JVM在运行时才动态加载类或调用方法/访问属性，不需要事先知道运行对象是谁
简单说，反射可以加载运行时才得知名称的.class文件，获悉完整结构，并生成对象实体，或对fields变量设值，或调用methods

### <span id="reflect_1">获取Class对象</span>
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

### <span id="reflect_2">创建实例</span>
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
### <span id="reflect_3">获取方法</span>
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

### <span id="reflect_4">获取类的成员变量</span>
* `getField`： 访问公有的成员变量
* `getDeclaredField`: 所有已声明的成员变量，但不能得到其父类的成员变量

### <span id="reflect_5">调用方法</span>
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
### <span id="reflect_6">访问私有方法</span>
使用`setAccessible(true)`,获取访问权；如果不设置，会报错
`java.lang.IllegalAccessException: Class MainClass can not access a member of class `

### <span id="reflect_7">修改私有常量</span>
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
### <span id="reflect_8">进阶</span>
2020.04.24
最近项目开发中，需要用反射技术，要反射的有类，静态函数，接口以及kotlin中的函数表达式。以前没有用到过，一步一个坑的进行调试。
#### <span id="reflect_8_1">静态函数</span>
反射执行`static`方法时，invoke第一个参数传递null
```java
//以单例模式为例
class Test{
    private Test(){}
    private static Test mInstance = new Test();
    public static Test getInstance(){
        return mInstance;
    }
    public void test(){
        Log.e("tag"," this is a test method");
    }
    public void testCallback(TestCallback callback){
//        callback.hello();
        Log.e("tag"," this is test callback");
        callback.callback();
    }
}
class ReflectDemo{
    public static void reflect(){
        try{
            Class cla = Class.forName("com.jerry.Test");
            Method method = cla.getDeclaredMethod("getInstance");
            method.setAccessible(true);
            //反射执行static方法时，invoke参数传递null，得到对象实例
            Object o = method.invoke(null);
    
            Method test = cla.getDeclaredMethod("test");
            test.invoke(o);
        }catch(Exception e){
            e.printStackTrace();
        }
    }
}
```
#### <span id="reflect_8_2">反射接口 —— 动态代理</span>
接口也可以反射，需要使用proxy创建代理类，完成接口的实例化。
```java
public interface TestCallback {
    void callback();
    String hello();
}
class ReflectDemo {
    public static void reflectCallback(){
        try{
            Class cla = Class.forName("com.jerry.Test");
            Method instance = cla.getDeclaredMethod("getInstance");
            instance.setAccessible(true);
            Object o = instance.invoke(null);
            
            //reflect interface
            Class claCallback = Class.forName("com.jerry.TestCallback");
            //动态代理
            final Object obj = Proxy.newProxyInstance(ReflectDemo.class.getClassLoader(), new Class[]{claCallback},new InvocationHandler() {
            @Override
            public Object invoke(Object o, Method method, Object[] objects) throws Throwable {
                Log.e("tag", " invoke callback ");
                return null;
                }
            });
        Method test = cla.getDeclaredMethod("testCallback", new Class[]{claCallback});
        test.setAccessible(true);
        test.invoke(o, new Object[]{obj} );
    }catch(Exception e){
            e.printStackTrace();
        }
    }
}

```
#### <span id="reflect_8_3">kotlin中函数类型作为参数的反射</span>
函数的参数为函数类型，通过反射Class对象，获取所有方法的信息，通过`method.getParameterTypes()`获取到参数的Class对象数据，遍历数组，打印参数的类型。
```kotlin
class AccountDemo {
    fun getInfo(callback: (Boolean, List<String>?)->Unit)?) {
        ...
        callback?.invoke(false, listOf("abc"))
        ...
        callback(true, listOf("ABC"))
    }
}
```
```java
public static void reflectAccount(){
    Class clazz = Class.forName("com.jerry.AccountDemo");
    Method[] declaredMethods = clazz.getDeclaredMethods();
    for (Method m : declaredMethods) {
    //获取参数数组
        Class<?>[] types = m.getParameterTypes();
        //遍历数组，获取参数类型
        for (Class<?> type : types) {
            Log.e("tag", "args type ="+type.getName());
        }
    }
    //kotlin中函数是Function类型，参数个数不同类型名称不同；得到kotlin.jvm.functions.Function2
    Class<?> funClass = Class.forName("kotlin.jvm.functions.Function2");
    //动态代理
    Object proxyFun = Proxy.newProxyInstance(RelectDemo.class.getClassLoader(),new Class[]{funClass}, new InvocationHandler(){
        @Override
        public Object invoke(Object o, Method method, Object[] objects) throw Throwable {
            Log.e("tag", "invoke kotlin callback");
            return null;
        }
    });
    Method method = clazz.getDeclaredMethod("getInfo", funClass);
    method.setAccessible(true );
    method.invoke(clazz.newInstance(), new Object[]{ proxyFun});
}

```
### <span id="reflect_9">反射android app中application</span>

反射application遇到的问题， 网上查了资料看源码，application正常启动是在`Instrumentation`中,代码如下，
```java
public static Application newApplication(Class<?> clazz, Context context) throws ClassNotFoundException,IllegalAccessException, InstantiationException{
//调用newInstance获取application实例对象，通过多态调用application中的attach函数;
    Application app = (Application)clazz.newInstance();
    app.attach(context);
    return app;
}
```
反射application遇到的坑， `attach`在Application中时用@hide修饰的，可见行是`package`; final的，不可被继承；源码如下
```java
/**
* @hide
*/
/* package */ final void attach(Context context) {
       attachBaseContext(context);
       mLoadedApk = ContextImpl.getImpl(context).mPackageInfo;
}
```
说下我遇到的问题，
1. 反射了app的application，获取函数列表时，没有`attach`函数；`getDeclaredMethods`和`getMethods`都试过；曾以为是hide修饰，所以获取不到；其实不是，hide修饰，只是生成javadoc时，隐藏了该方法而已。
真正的原因是反射的Class对象不对，应该反射Application，通过Application的class对象执行`getDeclaredMethods()`获取函数列表，此时就可以看到有attach方法名了。

1. 如何执行app中自定义Application的`oncreate`函数,  
获取到attach函数后，模拟源码执行方式，但是执行attach时，还是会报错，提示InvocationTargetException, 经过排查后，下面是正确的调用方式

反射自定义application，执行oncreate
```java
public void reflectApp(){
    Class clazz = Class.forName("android.app.Application");
    //获取attach方法，必须用getDeclaredMethod
    Method attach = clazz.getDeclaredMethod("attach", new Class[]{Context.class});
    Class coreClass = Class.forName("com.jerry.MyApplication");
    //模拟源码执行， 利用多态
    Application app = (Application) coreClass.newInstance();
    attach.setAccessible(true);
    attach.invoke(app, context);
    
    //反射获取并执行oncreate， 
    Method method = coreClass.getMethod("onCreate");
    method.setAccessible(true );
    method.invoke(app);
}
//注意：两次`invoke`传递的必须是同一个对象，否则会报错,执行失败.
```
至此可以说，没有反射干不了的事情，android源码功能轻松调用执行。


