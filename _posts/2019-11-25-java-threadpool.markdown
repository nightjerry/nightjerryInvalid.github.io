---
layout: post
title:  "java 线程池"
date: 2019-11-25 16:33:02 +0800
categories: jekyll update
tags: java
---

## 线程池ThreadPool
 
 [TOC]
 
 线程是一种昂贵的资源，开销有以下几种：

* 线程的创建与启动的开销. 与普通的对象相比，线程还占用了额外的存储空间 —— 栈空间； 线程的启动回产生线程调度开销
* 线程的销毁
* 线程调度的开销. 线程调度会导致上下文切换，增加处理器资源的消耗，减少了应用程序本身可以使用的处理器资源
* 线程的数量受限于系统拥有的处理器数目，上限等于处理器的数目

---------

### 线程工厂ThreadFactory
创建线程的工厂方法,`ThreadFactory`接口是工厂方法模式的一个实例；

```java
public Thread newThread(Runnable r)
//用于创建线程，r代表线程需要执行的任务
```

### 线程池执行状态分析

线程池内部预先创建一定数量的工作者线程，客户端讲需要执行的任务作为对象提交给线程池，线程池可能讲这些任务缓存在工作队列中，线程池内部的各个工作者线程则从队列中取出任务并执行；

线程池可看作是生产者——消费者模式的一种服务
内部维护的工作者线程相当于消费者线程；
线程池的客户端线程相当于生产者线程；
线程池内部缓存队列相当于传输通道；
客户端提交给线程池的任务相当于‘产品’;

线程池`java.util.concurrent.ThreadPoolExecutor`，`ThreadPoolExecutor.submit`提交任务

```java
public Future<?> submit(Runnable task)
//task代表客户端需要线程池执行的任务
```

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
//corePoolSize : 指定线程池核心线程数
//maximumPoolSize : 线程池最大线程数
//keepAliveTime，unit一起使用，指定线程池空闲(Idle)线程的最大存活时间
//workQueue : 工作队列的阻塞队列，相当于传输通道
//threadFactory : 用于创建工作者线程的线程工厂
//handler: 
```

1. 初始状态下，客户端每提交一个任务，线程池创建一个工作者线程来处理；
2. 当创建的线程池数量**等于**核心线程池大小的时候，新来的任务会存入工作队列之中；由线程池中的工作者线程负责取出执行。线程池将任务存入工作队列执行的**BlockingQueue.offer(E e),是非阻塞方法**[^a] ，工作队列满不会使客户端线程暂停;
3. 当工作队列满时，线程池会创建新的工作者线程，知道当前线程池达到最大线程池大小；线程池通过`threadFactory.newThread`创建工作者线程，如果创建线程池时没有指定线程工厂，会使用`Executors.defaultThreadFactory()`返回的默认线程
4. `线程池饱和(Saturated)`,即工作者队列满且线程池达到最大线程数的情况下，客户端试图提交的任务会被**拒绝Reject**; 为提高线程池的可靠性，`RejectedExecutionHandler`接口用于封装被拒绝的任务的处理策略，其接口方法如下

```java
void rejectedExecution(Runnable r, ThreadPoolExecutor executor)
//r : 被拒绝的任务
//executor : 拒绝任务r的线程池
```
> `RejectedExecutionHandler`可通过ThreadPoolExecutor构造函数设置，或者方法`setRejectedExecutionHandler()`

ThreadPoolExecutor提供了RejectedExecutionHandler的实现类，如果无法满足需求，可以自行实现

| 实现类 | 所实现的处理策略 |  
| --- | --- | --- |
| ThreadPoolExecutor.AbortPolicy | 直接抛出异常 |  |
| ThreadPoolExecutor.DiscardPolicy | 丢弃当前被拒绝的任务(不抛出异常) |  |
| ThreadPoolExecutor.DiscardOldestPolicy | 将工作队列中最老的任务丢弃，然后重新尝试接纳被拒绝的任务 |  |
| ThreadPoolExecutor.CallerRunsPolicy | 在客户端线程中执行被拒绝的任务  |  

5 当前线程池大小**超过**线程池核心大小时，超过线程池核心大小的工作者线程空闲(即队列中没有待处理任务)时间达到keepAliveTime所指定的时间后，就会被清理掉(从线程池中移除)。空闲清理机制有利于节约有限的线程资源；不合理设置keepAliveTime(特别是设置太小)会导致线程的频繁清理和创建，反而增加了开销

> 核心线程是逐渐被创建与启动的，使用`ThreadPoolExecutor.prestartAllCoreThreads()`使线程池在未接收到任何任务的情况下预先创建并启动，减少任务被线程池处理时所需的等待时间


### 线程池的关闭

`ThreadPoolExecutor.shutdown() / shutdownNow()`用来关闭线程池
`shutdown`:已提交的任务会被继续执行，新提交的任务会像线程池饱和时那样被拒绝;`shutdown`返回的时候线程池**可能**尚未关闭(即线程池中可能有还有工作者线程正执行任务)，可使用`ThreadPoolExecutor.awaitTermination(long timeout, TimeUnit unit)`等待线程池关闭结束
`shutdownNow`: 正在执行的任务会被停止(通过`Thread.interrupt`停止)，队列中的任务不会被执行；该方法返回值时已提交而未执行的任务列表

### 任务的处理结果
使用`ThreadPoolExecutor.submit(Runnable)`不关心任务的处理结果；如果客户端关心处理结果，可使用`ThreadPoolExecutor.submit(Callable<T> task)`

```java
public <T> Future<T> submit(Callable<T> task)
```
`java.util.concurrent.Callable`相当于增强型的Runnable接口，其唯一方法声明如下，

```java
V call() throws Exception
```
> call方法的返回值代表相应任务的处理结果，在执行中可以抛出异常；而Runnable接口的run方法既无返回值也不能抛异常; `Executors.callable(Runnable task, T result)`能将Runnable接口转为Callable接口实例

`submit`的返回值类型为`java.util.concurrent.Future`, `Future.get()`用来获取task参数所指定的任务的处理结果，

```java
V get() throws InterruptedException, ExecutionException
```

### 线程池死锁
线程池中执行任务时，向**同一线程池**提交另一个任务，而前一个任务的执行结束依赖于后一个任务的执行结果，后一个任务的执行又依赖前一个任务执行结果，导致死锁

> 同一线程池只能用于执行相互独立的任务；有依赖关系的任务需要提交不同的线程池执行以避免死锁

## ThreadLocal
`ThreadLocal`是线程内部的数据存储类，可以在指定的线程中存储数据，只有指定线程中可以获取。

```java
private ThreadLocal<Boolean> mBoolean = new ThreadLocal<Boolean>();

mBoolean.set(true);
Log.d(TAG,"[Thread#main]mBoolean="+mBoolean.get());

new Thread("Thread#1") {
    public void run(){
        mBoolean.set(false);
        Log.d(TAG,"[Thread#1]mBoolean="+mBoolean.get());
    }
}.start();

new Thread("Thread#2") {
    public void run(){
        Log.d(TAG,"[Thread#2]mBoolean="+mBoolean.get());
    }
}.start();
```
执行结果:

```
[Thread#main]mBoolean=true
[Thread#1]mBoolean=false
[Thread#2]mBoolean=null
```
`ThreadLocal`内部有一个数组`Object[] table`,`ThreadLocal`的值就存在这个table中，

## synchronized
`synchronized`是内部锁，能保障原子性，可见性和有序性。

## volatile
关键字`volatile`用来修饰成员变量，告知程序任何对该变量的访问需要从共享内存中获取，对变量的改变必须同步刷新到共享内存,保证所有线程对变量访问的可见性.

保障可见性，有序性，保障`long/double`型变量的原子性
单例模式中，DCL分析

```java
private static DCLSingletion instance = null;
public static DCLSingletion getInstance() {
    if (null == instance) { //操作1
        synchronized (DCLSingletion.class){
            if (null == instance) { //操作2
                instance = new DCLSingletion();//操作3
            }
        }
    }
}
```
在操作3中，可以分解为几个独立的子操作

```java
objRef = allocate(DCLSingletion.class);//子操作1:分配对象所需的存储空间
invokeConstructor(objRef);//子操作2:初始化objRef引用的对象
instance = objRef; //子操作3:将对象应用写入共享变量
```
临界区内的操作可以被冲排序，JIT编译器可能将上述的子操作重排序为：子操作1 -> 子操作3 -> 子操作2

> 一个线程执行操作1时发现instance不为null，该线程直接返回instance变量所引用的实例，而这个实例可能时未出初始化完毕的，这就可能导致程序出错；
用`volatile`关键字修饰instance即可解决此问题



[^a]: 阻塞/非阻塞是从CPU消耗上来看的
    阻塞就是CPU停下来等待一个慢的操作完成CPU才能做其他事情；
    非阻塞就是耗时任务执行时，CPU去做其他事；当耗时任务完成了，CPU再接着完成后续的操作；
    例如：堵车，阻塞就是司机什么也不做，专注跟车；非阻塞就是前车不动时，司机看报玩手机；前面开走了，司机继续开车。

