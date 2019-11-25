---
layout: post
title:  "go6 异常"
date: 2019-11-25 17:15:02 +0800
categories: jekyll update
tags: go
---

## 6 异常
[TOC]

#### panic 
抛出异常，终止程序

```go
func main(){
    fmt.Println("abc")
    panic("bcd")
    fmt.Println(123)
}
//abc 
//panic: bcd
```

#### error
```go
func main() {
	num, err := TestError(10, 0)
	if err != nil {
		fmt.Println(err)
	}else{
		fmt.Println(num)
	}
}

func TestError(num1 int, num2 int)(result int, err error){
	if num2 == 0{
		err = errors.New("除数不能为0")
		return
	}
	result = num1/ num2
	return
}
```
#### recover 错误拦截
> `recover`只有在`defer`调用的函数中有效, 
> defer不能直接作用在recover函数上

```go
func main() {
	defer Test()
	//defer recover() 直接修饰无效
	var x = 10
	var y = 0
	n := x /y
	fmt.Println(n)
}
func Test(){
	//fmt.Println(recover()) //打印错误信息
	recover()
}
```
### 文件
#### 创建文件
```go
import (
	"fmt"
	"os"
)
func main() {
	file, err := os.Create("a.txt")
	if err != nil {
		fmt.Println(err)
	}else{

	}
	defer file.Close()
}
```
#### 写入数据
```go
n, err := file.WriteString("Hello world")
//n 为写入数据长度
```
```go
var str = "Hello World"
n, err := file.Write([]byte(str))
```
```go
num,_ := file.Seek(0, io.SeekEnd)//io.SeekEnd将光标定位到文件内容末尾
//指定位置写入数据
n, err := file.WriteAt([]byte(str), num)
```
#### OpenFile()
```go
func OpenFile(name string, flag int, perm FileMode) (*File, error)
name: 文件的路径
flag: 模式; O_RDONLY(只读模式),O_WRONLY(只写模式),O_RDWR(可读可写),O_APPEND(追加模式)
perm: 权限,取值(0-7)
    0: 没有任何权限
    1: 执行权限
    2: 写权限
    3: 写权限与执行权限
    4: 读权限
    5: 读权限与执行权限
    6: 读权限与写权限
    7: 读权限, 写权限, 执行权限
```
```go
import (
	"fmt"
	"os"
)

func main() {
	file, err := os.OpenFile("a.txt", os.O_APPEND|os.O_WRONLY, 6)
	if err != nil {
		fmt.Println(err)
	}
	defer file.Close()
	n, err := file.WriteString("ccc")
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(n)
}
```
#### 读取文件
`os.Open()` == `Open(name, O_RDONLY, 0)`

```go
func Open(name string) (*File, error) {
	return OpenFile(name, O_RDONLY, 0)
}
```
```go
func main(){
    file, err:=os.Open("a.txt")
	if err != nil {
		fmt.Println(err)
	}
	defer file.Close()

	buffer := make([]byte, 1024*2)
	n, err := file.Read(buffer)
	if err != nil {

	}
	fmt.Println(n)
	fmt.Println(string(buffer[:n])) //文件内容
}
```
循环读取

```go
func main(){
    file, err:=os.Open("a.txt")
	if err != nil {
		fmt.Println(err)
	}
	defer file.Close()

	buffer := make([]byte, 10)
	for {
	   n, err := file.Read(buffer)
	   if err == io.EOF{
         //文件结束
	       break
	   }
	   fmt.Println(n)
	   fmt.Println(string(buffer[:n])) //文件内容
	}
	
}
```
按行读取

```go
//创建带有缓冲区的Reader
reader := bufio.NewReader(file/*打开的文件指针*/) 
buf, err := reader.ReadBytes('\n')  //读取长度取决于参数
//判断文件结尾
if err != nil && err == io.EOF
```
读取目录项

```go
func main(){
    file, err := os.OpenFile(path, os.O_RDONLY, os.ModeDir)
    if err != nil {}
    defer file.Close()
    fileInfo,err := file.Readdir(-1)
    for _, info := range fileInfo {
        if info.IsDir(){ 
            //dir
        }else{
            //file
        }
    }
}
```

### 字符串string
```go
bool := strings.HasSuffix("hello.txt", ".txt")
//判断是否是`.txt`结尾
```
```go
var str = "hello"
b := strings.Contains(str, "he")
//str是否包含字串
```
```go
//字符串拼接
s := []string{"hello", "world","go"}
str := strings.Join(s, "-")//使用"-"拼接切片元素
fmt.Println(str)
//hello-world-go
```
```go
//查找子串在字符串中的位置
str := "helloworld"
//查找"llo"在str中的位置
n :=strings.Index(str, "llo")
```
```go
//表示"go"重复3次
str := strings.Repeat("go",3)
fmt.Println(str) //gogogo 
```

```go
func Replace(s, old, new string, n int) string
// 原字符串， 旧字符串，新字符串，替换个数(-1:代表全部)
```
```go
//Replace: 替换
str := "aabbcc"
s := strings.Replace(str, "a" ,"A", 1)
fmt.Println(s) //Aabbcc
```
```go
//split分割
str := "hello@go@abc"
s := strings.Split(str, "@")//得到切片
fmt.Println(s) //[hello go abc]
```
#### 字符串转换
通过`strconv`实现

```go
str := strconv.FormatBool(true) 
s := strconv.Itoa(123) //int to string
```
将字符串转换其他类型

```go
b, err := strconv.ParseBool("true")
num, err := strconv.Atoi("123")
```
### 文件压缩


