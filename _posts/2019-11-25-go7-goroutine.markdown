---
layout: post
title:  "go7 并发"
date:   2019-11-25 17:17:02 +0800
categories: jekyll update
tags: go
---

## 7 并发
[TOC]
### 并行 (parallel)
同一时刻，有多条指令在多个处理器上同时执行; 借助多核实现
并发 : 微观：

### Go并发
通过两种方式实现并发,`goroutine`及`channel`

```go
func main()  {
	 go sign()
	dance()
	 //for {;}
}

func sign(){
	for i:=0; i< 5 ; i++ {
		fmt.Println("is singing---", i)
		time.Sleep(100 * time.Millisecond)
	}
}
func dance(){
	for i:= 0; i< 5; i++ {
		fmt.Println("is dancing",i)
		time.Sleep(100 * time.Millisecond)
	}
}
is dancing 0
is singing--- 0
is dancing 1
is singing--- 1
is singing--- 2
is dancing 2
is dancing 3
is singing--- 3
is dancing 4
is singing--- 4
```
```go
func main(){
//另一种创建子go程方式
    go func(){
        fmt.Println("this is goroutine")
    }()
    for{;}
}
```
> 主go程结束，子go程随之退出

#### runtime.Gosched()
出让当前go程所占用的cpu时间片,让出当前goruntine的执行权限;
调度器安排其他等待的任务运行，并在下次获取cpu时间轮片时，从Gosched的位置恢复执行

#### runtime.Goexit()
调用Goexit将立即终止当前goruntine,调度器会确保所有已注册的defer延迟调用被执行
> 只能在子go程使用

```go
func test(){
    defer fmt.Println("aaa")
    runtime.Goexit()
    fmt.Println("bbb")
}
func main(){
    go func(){
        fmt.Println("ccc")
        test()
        fmt.Println("ddd")
    }()
    for {;}
}

ccc
aaa
```
#### runtime.GOMAXPROCS()
用来设置可以并行计算的CPU核数的最大值，并返回之前的值

```go
func main(){
    n := runtime.GOMAXPROCS(1) //设置1核
    
}
```
`runtime.GOROOT()`

### channel
一种数据类型，对应一个管道，FIFO
> 用于解决go程的同步问题及协程之间数据共享(数据传递)问题
> goroutine通过通信来共享内存，而不是共享内存来通信

定义: `make(chan 数据类型, 容量) `
 容量=0 :无缓冲;
 容量>0 : 有缓冲channel

```go
make(chan int)
make(chan string, 0)
```
channel有两个端:

+ 写端(传入端): `chan <-`
+ 读端(传出端): `<- chan`

> 读端和写端必须**同时**满足条件，才在chan上进行数据流动，否则，阻塞
> 读端读数据，同时写端不在写，读端阻塞
> 写端写数据，读端不读，阻塞
> **读写端不能在同一个go程中**

#### channel数据通信
```go
func main(){
	ch := make(chan string)

	ch <- "打印"
	go func() {
		fmt.Println("子go程执行")
	}()
}
出现死锁
//fatal error: all goroutines are asleep - deadlock!
```
```go
func main(){

	ch := make(chan string)
	//len(ch)：channel中剩余未读取数据个数
	//cap(ch): 通道的容量
	fmt.Println("len(ch)=",len(ch), "cap(ch)=",cap(ch))
	go func() {
		fmt.Println("子go程执行")
		ch <- "子go程执行完毕"
	}()
	str := <-ch
	fmt.Println(str)
}
```
#### 无缓冲channel
`同步`: 在两个或多个协程(线程)间，保持数据内容一致性的机制
`阻塞`: 数据没有到达，当前协程(线程)持续处于等待状态,直到条件满足,才解除阻塞

无缓冲channel创建格式:
`make(chan Type) `
`make(chan Type, 0)`
通道容量为0，len=0; 不能存储数据
具备同步能力，读写同步;

```go
func main(){
	ch := make(chan int)

	go func() {
		for i:=0; i< 5; i++{
			fmt.Println("子go程执行", i)
			ch <- i
		}
	}()
	for i:=0;i <5;i++{

		str := <-ch
		fmt.Println("主go程读",str)
	}
}
子go程执行 0
子go程执行 1
主go程读 0
主go程读 1
子go程执行 2
子go程执行 3
主go程读 2
主go程读 3
子go程执行 4
主go程读 4
```
#### 有缓冲channel
缓冲区可以进行数据存储, 存储至容量上限才阻塞;
具备异步通信能力, 不需要同时操作channel缓冲区

```go
func main(){
	ch := make(chan int, 2)

	fmt.Println("len(ch)",len(ch), "cap(ch)", cap(ch))
	go func() {
		for i:=0; i< 5; i++{
			ch <- i
			fmt.Println("子go程,", i,"len(ch)",len(ch), "cap(ch)", cap(ch))
		}
	}()
	for i:=0; i< 5;i++ {
		num := <-ch
		fmt.Println("主go程读取", num)
	}

}
len(ch) 0 cap(ch) 2
子go程, 0 len(ch) 0 cap(ch) 2
子go程, 1 len(ch) 1 cap(ch) 2
子go程, 2 len(ch) 2 cap(ch) 2
主go程读取 0
主go程读取 1
主go程读取 2
主go程读取 3
子go程, 3 len(ch) 2 cap(ch) 2
子go程, 4 len(ch) 0 cap(ch) 2
主go程读取 4
```
#### 关闭channel
`close(ch)`
另一端可以判断channel是否关闭
`num,ok := <-ch ; ok ==true`
ok = true, channel 未关闭, num保存读到的数据
ok = false: channel 关闭,num无数据
确定不向对端发送(接收)数据时，关闭channel; 通常关闭写入端

```go
func main(){
	ch :=make(chan int)

	go func(){
		for i:=0; i<5; i++{
			ch <- i
		}
		close(ch)//写端写完数据，主动关闭数据
	}()

	for{
		if num, ok := <-ch; ok == true{
			fmt.Println("read data = ", num)
		}else {
			fmt.Println("channel is close")
			break
		}
	}
}
read data =  0
read data =  1
read data =  2
read data =  3
read data =  4
channel is close
```
> 数据没有写入完毕，不应该关闭channel
> 已经关闭的channel，不能再向其写入数据//panic: send on closed channel
> 已经关闭的channel, 可以读取数据; 读到0

```go
//向已关闭channel写入数据
func main(){
	ch :=make(chan int)

	go func(){
		for i:=0; i<5; i++{
			ch <- i
		}
		close(ch)//写端写完数据，主动关闭数据
		ch <- 11
	}()

	for{
		if num, ok := <-ch; ok == true{
			fmt.Println("read data = ", num)
		}else {
			fmt.Println("channel is close")
			break
		}
	}
	for{;}
}
//panic: send on closed channel
```
读无缓冲channel: 读到0 说明写端关闭
读有缓冲channel: 如果缓冲区内有数据，先去数据;读完数据后，可以继续读, 读到0

```go
//for range读取channel
func main(){
	ch :=make(chan int, 3)

	go func(){
		for i:=0; i<5; i++{
			ch <- i
		}
		close(ch)//写端写完数据，主动关闭数据
	}()
	for num := range ch {
		fmt.Println("read data ", num)
	}
}
```
#### 单向channel
默认的channel是双向的  
定义: `var ch chan int ` 
赋值: `ch = make(chan Type)`

单向写channel: 
定义: `var sendCh chan<- int`
赋值: `sendCh = make(chan<- int)`

单向读channel: `var readCh <-chan int`
赋值: `readCh = make(<-chan int)`

转换: 

+ 双向channel可以隐式转换为任意一种单向channel `sendCh = ch`
+ 单向channel不能转换为双向channel

#### 单向channel作函数参数
函数中传参: 传引用

```go
func main() {
	ch := make(chan int)
	go func() {
		send(ch)//双向channel转为写channel
	}()
	recv(ch)
}
func send(out chan<- int){
	out <- 123
	close(out)
}
func recv(in <-chan int){
	num := <-in
	fmt.Println("read data ", num)
}
```
```go
//channel用于go程通信
var ch =make(chan int)
func main() {

	go person1()
	go person2()
	for{;}

}
func printer(s string){
	for _, ch := range s{
		fmt.Printf("%c", ch)
		time.Sleep(200 * time.Millisecond)
	}
}
func person1(){
	printer("hello")
	ch <- 9
}
func person2(){
	<- ch
	printer("world")
}
helloworld
```


