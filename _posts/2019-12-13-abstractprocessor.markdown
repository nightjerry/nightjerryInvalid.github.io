---
layout: post
title:  "EventBus源码分析之AbstractProcessor"
subtitle:   "注解处理器"
date:   2019-12-13 10:20:00 +0800
author: jerry
header-img: "img/avatar.jpg"
categories: jekyll update
tags: [java android]
---
# EventBus源码分析之AbstractProcessor

[TOC]  

## AbstractProcessor 注解处理器
目前很多Android库都是使用注解的方式实现的.处理注解分两种，   
- 运行时(Runtime)通过反射机制运行处理的注解，  
- 编译时(Compile time)通过注解处理器处理的注解

### AbstractProcessor简介
注解处理器是javac中的，在编译时扫描和处理注解的工具
可以生成java代码，组成.java文件
抽象处理器`AbstractProcessor`,每个注解处理器都要继承它
```java
import java.util.Set;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.ProcessingEnvironment;
import javax.annotation.processing.RoundEnvironment;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.TypeElement;

public class MyProcessor extends AbstractProcessor {

    /**
     * 特殊的init()方法，它会被注解处理工具调用，并输入ProcessingEnviroment参数。
     * @param processingEnvironment 提供很多有用的工具类Elements,Types和Filer
     */
    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
    }

    /**
     * 相当于每个处理器的主函数main()。
     * 在这里写你的扫描、评估和处理注解的代码，以及生成Java文件
     * @param set
     * @param roundEnvironment 可以查询出包含特定注解的被注解元素
     * @return
     */
    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        return false;
    }

    /**
     * 注解处理器是注册给哪个注解的
     * 等同于@SupportedAnnotationTypes("注解类全路径")
     * @return 包含本处理器想要处理的注解类型的合法全称(包名.类名)
     */
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        return super.getSupportedAnnotationTypes();
    }

    @Override
    public Set<String> getSupportedOptions() {
        return super.getSupportedOptions();
    }

    /**
     * 使用的Java版本。通常返回SourceVersion.latestSupported()。
     * 如有指定版本，如Java7，返回SourceVersion.RELEASE_7
     * 通过@SupportedSourceVersion可以代替此方法
     * @return
     */
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return super.getSupportedSourceVersion();
    }

}
```
### 方法详解
`init`可以获取有用的工具类
```java
public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
        Types typeUtils = processingEnvironment.getTypeUtils();
        Elements elementUtils = processingEnvironment.getElementUtils();
        Filer filer = processingEnvironment.getFiler();
        Messager messager = processingEnvironment.getMessager();
    }
```
`Types` 提供了和类型相关的一些操作，如获取父类、判断两个类是不是父子关系等
`Elements` 提供了一些和元素相关的操作，如获取所在包的包名等
`Filer`用于文件操作,用它创建生成的代码文件
`Messager` 用于打印日志信息,可以打印出Element所在的源代码，也可抛出异常。靠默认的错误打印有时很难找出错误的地方,用它去添加更直观的日志打印

#### Element的子接口
`process ` 方法中使用`getElementsAnnotatedWith`获取到的都是Element接口,其实我们用Element.getKind获取到类型之后可以将他们强转成对应的子接口
- `TypeElement`：表示一个类或接口程序元素。
- `PackageElement`：表示一个包程序元素。
- `VariableElement`：表示一个属性、enum 常量、方法或构造方法参数、局部变量或异常参数。
- `ExecutableElement`：表示某个类或接口的方法、构造方法或初始化程序（静态或实例），包括注释类型元素。
```java
package com.example;    // PackageElement

    public class Test {      // TypeElement
        private int a;      // VariableElement
        private Foo other;  // VariableElement
        public Foo () {}    // ExecuteableElement
        public void setA (  // ExecuteableElement
                int newA    // TypeElement
        ) {}
    }
```
#### Element(s)常用的api
获取类名：
```java 
Element.getSimpleName().toString(); // 获取类名
Element.asType().toString(); //获取类的全名
```
获取所在的包名:
```java
Elements.getPackageOf(Element).asType().toString();
```
获取所在的类:
```java
Element.getEnclosingElement();
```
获取父类:
```java
Types.directSupertypes(Element.asType());
```
获取标注对象的类型:
```java
Element.getKind();
```

`@AutoService(Processor.class)` 是google开发的注解处理器，全路径为`com.google.auto.service.AutoService`,用来生成`META-INF/services/javax.annotation.processing.Processor`文件的

在gradle中引入,方式如下:
```
implementation 'com.google.auto.service:auto-service:1.0-rc6'
```
```java
@Target({ElementType.TYPE}) //定义在注解的作用域，类上
@Retention(RetentionPolicy.CLASS) //
public @interface MyAnnotation {
    String name() default "unkown";

    int age() default 0;

}
```
```java
@SupportedAnnotationTypes({"com.okay.processor.MyAnnotation"})
@AutoService(Processor.class)
public class MyProcessor extends AbstractProcessor {
    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
        Types typeUtils = processingEnv.getTypeUtils();
        mElementUtils = processingEnv.getElementUtils();
        mFiler = processingEnv.getFiler();
        mMessager = processingEnv.getMessager();

    }
    Elements mElementUtils;
    Messager mMessager;
    Filer mFiler;
    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        //getElementsAnnotatedWith: 返回所有被注解了@MyAnnotation的元素的列表
        Set<? extends Element> elements = roundEnvironment.getElementsAnnotatedWith(
                MyAnnotation.class);
        for (Element element : elements) {
            //检查被注解的元素是否是 类
            if (element.getKind() != ElementKind.CLASS) {
                mMessager.printMessage(Diagnostic.Kind.ERROR, String.format("only class can be " +
                                                                                    "annotated " +
                                                                                    "with @%s",
                                                                            MyAnnotation.class.getSimpleName()));
                return true;
            }
            analysisAnnotated(element);
        }
        return true;
    }
    
    private final String SUFFIX = "AUTO";
    //拼接类
    private void analysisAnnotated(Element element) {
        MyAnnotation annotation = element.getAnnotation(MyAnnotation.class);
        String name = annotation.name();
        int age = annotation.age();
        String newClassName = name+SUFFIX;
        StringBuilder sb = new StringBuilder()
                .append("package com.okay.processor.auto;\n\n")
                .append("public class ")
                .append(newClassName)
                .append(" {\n\n") //start class
                .append("\tpublic String getMsg() {\n") //start method
                .append("\t\treturn \"")  //start return
                .append(name).append(age)
                .append(".")
                .append("\";\n") //end return
                .append("\t}\n") //end method
                .append("}\n");    //end class

        writeFile(element,newClassName, sb.toString());
    }
    private void writeFile(Element element,String className, String content){
        String packageName = mElementUtils.getPackageOf(element).asType().toString();
 //       String packageName = "com.okay.processor.auto.";
        try {
            JavaFileObject sourceFile = mFiler.createSourceFile(packageName + className);
            Writer writer = sourceFile.openWriter();
                writer.write(content);
                writer.flush();
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.RELEASE_7;
    }
}
```