---
layout: post
title:  "算法之排序"
date:   2019-11-21 17:03:02 +0800
categories: jekyll update
tags: algorithms
---

[TOC]

### 堆栈
#### 栈(动态调整数组大小)

```java
import java.util.Iterator;
public class ResizingArrayStack<Item> implements Iterable<Item> {
    private Item[] a = (Item[]) new Object[1]; //栈元素
    private int N = 0;
    public boolean isEmpty() {
        return N == 0;
    }
    public int size(){ return N; }
    private void resize(int max) {
        Item[] tmp = (Item[]) new Object[max];
        for(int i=0; i< N; i++) {
            tmp[i] = a[i];
        }
        a = tmp;
    }
    
    public void push(Item item) {
        if (N == a.length) 
            resize(2* a.length);
        a[++N] = item;
    }
    public Item pop(){
        Item item = a[--N];
        a[N] = null; //避免对象游离
        if(N > 0 && N == a.length/4) 
            resize(a.length/2);
        return item;
    }
    public Iterator<Item> iterator(){
        return new ReverseArrayIterator();
    }
    private class ReverseArrayIterator implements Iterator<Item> { //支持后进先出的迭代
        private int i= N;
        public boolean hasNext(){ return i > 0;}
        public Item next(){ return a[--i]; }
        public void remove(){...}
    }
}

```

#### 链表
是一种递归的数据结构，或者为空，或者指向一个结点(node)的引用，该结点含有一个泛型的元素和一个只想另一条链表的引用.

#### 下压堆栈(链表实现)  

```java
public class Stack<Item> implements Iterable<Item> {
    private Node first; //栈顶
    private int N; //元素数量
    private class Node { //结点
        Item item;
        Node next;
    }
    
    public boolean isEmpty(){ return first == null;}
    
    public int size() {return N; }
    
    public void push(Item item) {
        Node oldfirst = first;
        first = new Node();
        first.item = item;
        first.next = oldfirst;
        N++;
    }
    
    public Item pop(){
        Item item = first.item;
        first = first.next;
        N--;
        return item;
    }
    
}
``` 
#### 队列(FIFO)

```java
public class Queue<Item> implements Iterable<Item> {
    private Node first; //最早添加结点的链接
    private Node last; //最后
    
    private int N; //队列中元素数量
    private class Node {
        Item item;
        Node next;
    }
    public boolean isEmpty() { return first == null; } // or N == 0
    public int size(){ return N; }
    public void enqueue(Item item) {//表尾添加元素
        Node oldLast = last;
        last = new Node();
        last.item = item;
        last.next = null;
        if(isEmpty())
            first = last;
        else
            oldLast.next = last;
        N++;
    }
    public Item dequeue(){ //表头删除元素
        Item item = first.item;
        first = first.next;
        if (isEmpty())
            last = null;
        N--;
        return item;
    }
    //TODO iterator()
}
```

#### 背包
一种不支持从中删除元素的集合数据类型，目的帮组用例收集元素并迭代遍历所有收集到的元素

```java
public class Bag<Item> implements Iterable<Item> {
    private Node first;
    private class Node {
        Item item;
        Node next;
    }
    public void add(Item item) {
        Node oldFirst = first;
        first = new Node();
        first.item = item;
        first.next = oldFirst;
    }
    public Iterator<Item> iterator(){
        return new ListIterator();
    }
    private class LastIterator implements Iterator<Item> {
        private Node current = first;
        public boolean hasNext() {
            return current != null;
        }
        public Item next(){
            Item item = current.item;
            current = current.next;
            return item;
        }
    }
}
```

```java
public class Example {
    //比较大小
    public static boolean less(Comparable v, Comparable w) {
        return v.compareTo(w) < 0;
    }
    //
    public static void exch(Comparable[] a, int i, int j) {
        //if (i == j) return;
        Comparable t = a[i];
        a[i] = a[j];
        a[j] = t;
    }
    //打印单行数组
    public static void show(Comparable[] a){
        for (int i=0;i < a.length;i++) {
            StdOut.print(a[i]+" ");
        }
        StdOut.println();
    }
    //测试数组元素是否有序
    public static boolean isSorted(Comparable[] a) {
        for (int i =0;i < a.length; i++) {
            if (less(a[i], a[i-1))
                return false;
        }
        return true;
    }
}
```
### 选择排序

```java
//双重for循环
public class Selection {
    public static void sort(Comparable[] a) {
    //将a[]按升序排列
        int N = a.length;
        //将a[i]和a[i+1 .. N]中最小的元素交换
        for (int i=0;i < N; i++) {
            int min = i;//最小元素索引
            //每轮内部for结束，得到一个最小值，并排序放入数组
            for (int j=i+1; j< N; j++) {
                if (less(a[j], a[min]))
                    min =j;
            }
            exch(a,i,min);
        }
    }
}

```
### 插入排序  
当前索引左边的所有元素都是有序的，对于部分有序的数组十分高效，适合小规模数组

```java
public class Insertion {
//升序排列
    public static void sort(Comparable[] a) {
    //将a[i]插入到a[i-1],a[i-2]...之中
        int n = a.length;
        for (int i=1; i< n; i++) {
        //每轮内部for结束，索引i前的部分是相对有序的
            for (int j =i; j> 0 && less(a[j],a[j-1]); j--) 
            //i与它之前的所有元素做对比，如果i小，则向前移动一位，继续与前一个比较；如果i大，则位置不变，停止内循环；
                exch(a,j,j-1);
        }
    }
}
```
选择排序不会访问索引**左侧**元素，插入排序不会访问索引**右侧**元素；
> 这些初级算法帮助我们建立了一些基本规则；
> 展示了性能基准；
> 某些特殊情况下它们也是很好的选择；
> 是开发更强大的排序算法的基石；

### 希尔排序
为加快速度简单的改进了**插入排序**
> 如果主键最小的元素在数组末尾，将它挪到正确的位置需要移动n-1次

交换不相邻的元素对数组的局部进行排序，用插入排序将局部有序的数组排序；
希尔排序的思想是使数组中任意间隔为h的元素都有序，这样的数组称为**h有序数组**

希尔排序高效的原因是: 
权衡了紫薯组的规模和有序性;排列之初，各个子数组都很短; 排序之后子数组都是部分有序的.

> 希尔排序实现思想：基于插入排序，并进行分组；（不稳定[^a]）
1，指定分组长度h，且有规则递减，终值为1(当h=1时，完成排序)
2，分组；方式：遍历，初始值从h开始，遍历从h至末尾的元素；目的是分组，分为h组；
3，针对分组排序；方式：循环遍历，初始值j从h开始，循环条件j大于等于h，且分组内相邻元素做比较；
    更新j：j-=h可以理解为普通for中的j--
4，更新h

```java
public class Shell{

    public static void sort(Comparable[] a){
        int n = a.length;
        int h= 1;
        while(h < n/3)
            h = 3*h+1;//指定子数组个数
            
        while(h>=1){
            //将数组变为h有序
            for(int i=h; i < n; i++){
                for( int j=i; j>= h && less(a[j],a[j-h]); j-= h){
                    exch(a, j, j-h);
                }
            }
            h= h/3;//递减h
        }
    }
}    
```
选择，插入，希尔排序不需要使用额外的内存空间。
希尔排序可以用于大型数组

### 归并排序
将两个有序的数组归并成一个更大的有序数组
需要额外的空间

#### 自顶向下的归并排序: 应用分治思想

```java
public class Merge {
/*
该方法将所有元素负责到aux[]中，然后再归并回a[]中；
归并时进行了4个条件判断：
1,左边用尽(取右边的元素)
2,右边用尽(取左边的元素)
3,右边的当前元素 < 左边的当前元素 (取右半边的元素)
4,右半边的当前元素>= 左半边的当前元素(取左半边的元素)
*/
    public static void merge(Comparable[] a, int lo, int mid, int hi){
        //将a[lo..mid]和a[mid+1..hi]归并
        int i=lo, j = mid+1;
        //复制数组
        for(int k=lo; k <= hi; k++)
            aux[k] = a[k];
            
        for(int k =lo; k<= hi; k++){
            if(i> mid) //a[lo..mid]已赋值
                a[k] = aux[j++];
            else if(j> hi) 
                a[k] = aux[i++];
            else if(less(aux[j],aux[i]))
                a[k] = aux[j++];
            else
                a[k] = aux[i++];
        }
    }
    
    private static Comparable[] aux;//辅助数组
    
    public static void sort(Comparable[] a){
        aux = new Comparable[a.length];
        sort(a, 0, a.length-1);
    }
    private static void sort(Comparable[] a, int start, int end){
        if (end <= start) return ;
        int mid = start + (end -start)/2;
        sort(a, start, mid); //将左半边排序
        sort(a, mid+1, end); //将右半边排序
        merge(a, start, mid, end);//归并排序
    }
}
```
> 注意：aux[i++]: 等同于aux[i] ,i++; 先取数组中i的值，再修改i；

```java
Integer[] a = {15,7,5,3,9,2,6,4};//length=8
sort(a,0,7); //mid=3;
--sort(a,0,3); //mid=1
    --sort(a,0,1);// mid = 0;
        --sort(a,0,0);
        --sort(a,1,1);
        --merge(a,0,0,1); //i=0,j=1; less(aux[j],aux[i]) j>hi
        [7,15,...]
    --sort(a,2,3);// mid = 2;
        --sort(a,2,2);
        --sort(a,3,3);
        --merge(a,2,2,3);//i=2,j=3; less[aux[j],aux[i])  j>hi
        [7,15,3,5,...]
    --merge(a,0,1,3); //i=0,j=2;  less() less() 
    [3,5,7,15,...]
            
--sort(a,4,7);// mid = 5;
    --sort(a,4,5); //mid=4
        --sort(a,4,4);
        --sort(a,5,5);
        --merge(a,4,4,5);//i=4,j=5; less() j> hi
        [3,5,7,15,2,9,..]
    --sort(a,6,7); //mid = 6;
        --sort(a,6,6);
        --sort(a,7,7);
        --merge(a,6,6,7);//i=6,j=7; less() j>hi
        [3,5,7,15,2,9,4,6]
    --merge(a,4,5,7);//i=4,j=6; else less less j>hi
    [3,5,7,15,2,4,6,9]
    
--merge(a,0,3,7); //i=0,mid=4; less()
[2,3,4,5,6,7,9,15]
```
归并排序思想：**递归**
1,将数组进行有穷两分，一分二，二分四，直到无法分割(数组最小长度为2)；
2,对每一份(两个元素)进行排序，然后四四排序，

#### 自底向上的归并排序
先归并那些微型数组，然后再成堆归并得到的子数组
步骤：
1,两两归并(每个元素理解为大小为1的数组)
2,四四归并(将大小为2的数组归并成大小为4的数组)
3,直到原数组有序

```java
public static void sort(Comparable[] a){
    int n = a.length;
    aux = new Comparable[n];
    for(int sz =1; sz<n; sz += sz)//sz子数组大小
        for(int lo=0; lo< n-sz; lo+= sz+sz)//lo子数组索引
            merge(a,lo,lo+sz-1, Math.min(lo+sz+sz-1,n-1));
}
```
自底向上的归并排序比较适合用链表组织的数据

### 快速排序
原地排序，分治的算法
快速排序的内循环比大多数排序算法都短小，意味着无论在理论上还是实际中都更快

快速排序和归并排序是互补的
归并排序：将数组分成两个子数组分别排序，并将有序的子数组归并以将整个数组排序；
快速排序：将数组分成两个子数组，将两部分独立的排序，当两个子数组都有序时，整个数组也就自然有序了

归并排序：递归调用发生在处理整个数组**前**；
快速排序：递归调用发生在处理整个数组**后**；

归并排序：数组被等分两半；
快速排序：切分(partition)的位置取决于数组的内容；

```java
public class Quick{
    public static void sort(Comparable[] a){
        sort(a, 0, a.length-1);
    }
    private static void sort(Comparable[] a, int lo, int hi){
        if (hi <= lo) return ;
        int j = partition(a, lo, hi);
        sort(a, lo, j-1);
        sort(a, j+1, hi);
    }
    
    private static int partition(Comparable[] a, int lo, int hi){
        //将数组切分为a[lo..i-1],a[i],a[i+1..hi]
        int i=lo, j=hi+1; //左右扫描指针
        Comparable v = a[lo]; //切分元素
        while(true){
            while(less(a[++i], v))
                if (i == hi)
                    break;
            while(less(v, a[--j]))
                if(j == lo)
                    break;
            if(i >= j)
                break;
            exch(a, i, j);
        }
        exch(a, lo, j);
        return j;
    }
}
```
>该算法的关键在于切分，这个过程使数组满足三个条件；
> 对于某个j,a[j]已经排定；
> a[lo]到a[j-1]中所有元素都不大于a[j];
> a[j+1]到a[hi]中所有元素都不小于a[j];

当指针i和j相遇时主循环退出;在循环中，`a[i]<v`时，`i++`;`a[j]>v`时，`j--`;交换a[i],a[j]来保证i左侧的元素都不大于v，j右侧的元素都不小于v

### 三向切分的快速排序
处理大量重复元素的数组
维护3个指针，
指针lt: a[lo..lt-1]中元素都小于v；
指针gt: a[gt+1..hi]中的元素都大于v；
指针i: a[lt..i-1]中的元素都等于v；
a[i..gt]的元素待排序

```
a[i] < v ,交换a[lt],a[i]; 然后lt++， i++;
a[i] > v, 交换a[gt],a[i]; 然后gt++;
a[i] = v, 将i++;
```
```java
public class Quick3way{
    private static void sort(Comparable[] a, int lo, int hi){
        if (hi <= lo) return;
        int lt = lo, i = lo+1, gt=hi;
        
        Comparable v = a[lo];
        while(i <= gt){
            int cmp = a[i].compareTo(v);
            if(cmp <0)
                exch(a, lt++, i++);
            else if(cmp >0)
                exch(a, i, gt--);
            else
                i++;
        }//现在a[lo..lt-1] < v =a[lt..gt] < a[gt_1..hi]
        sort(a, lo, lt-1);
        sort(a, gt+1, hi);
    }
}
```
>a[i]小于v，将a[lt]和a[i]交换，将lt和i加一；
>a[i]大于v，将a[gt]和a[i]交换，将gt减一；
>a[i]等于v，将i加一；

### 优先队列
堆排序是基于堆的优先队列的实现
重要的操作是**删除最大元素**和**插入元素**

`堆`：当一颗二叉树的每个节点都**大于等于**它的两个子节点时，称为**堆有序**

**二叉堆**是一组能能够用有序的完全二叉树排序的元素，并在数组中按照层级存储(不使用数组的第一个位置)

> 注意：
> 在一个(二叉)堆中，位置k的结点的父结点的位置为**k/2**
> 它的两个子节点的位置则分别为2k和2k+1；
> 从a[k]向上一层就令k等于k/2,向下一层则令k等于2k或2k+1

堆的算法：
用长度N+1的数组pq[]来表示大小为N的堆，不实用pq[0],堆元素放在pq[1]至pq[N]中  

```java
private boolean less(int i, int j){
    return pq[i].compareTo(pq[j]) < 0;
}
private void exch(int i, int j){
    Key t = pq[i];
    pq[i] = pq[j];
    pq[j] = t;
}
```

**由下至上的堆有序化(上浮)**
由于某个结点变得比它的父节点更大而打破(或堆底加入新的元素时)，需要交换它和它的父节点来修复堆；将这个节点不断向上移动直到我们遇到一个更大的父节点

```java
private void swim(int k){
    while(k >1 && less(k/2,k)){
        exch(k/2, k);
        k = k/2; 
    }
}
```

**由上至下的堆有序化(下沉)**
堆的有序状态因为某个节点变得比它的两个子节点或是其中一个更小而打破，将它和它的**`两个子节点中的较大者[1]`**交换来恢复堆；将节点向下移动直到它的子节点都比它更小或到达堆的底部。

```java
private void sink(int k){
    while(2*k <= N) { //k的下一层下标为2k，2k+1，这里防止角标越界
        int j = 2*k;
        if (j < N && less(j, j+1)) //见上[1]
            j++;
        if(!less(k, j))//如果根节点k不小于子节点j，则结束
            break;
        exch(k, j); //根节点小于子节点，与子节点交换
        k = j;
    }
}
```
**插入元素**：将新元素加到数组末尾，增加堆的大小，并让新元素上浮到合适的位置；
**删除最大元素**：从数组顶端删除最大的元素并将数组的最后一个放到顶端(*即：末尾与顶端的元素位置互换*)，减小堆的大小，并让该元素下沉到合适的位置；

> * **插入元素使用上浮，删除元素使用下沉**

#### 基于堆的优先队列
优先队列由一个基于堆的完全二叉树表示，存储于数组**pq[1..N]**中，pq[0]没有使用

```java
public class MaxPQ<Key extends Comparable<Key>> {
    private Key[] pq; //基于堆的完全二叉树
    private int N = 0;
    
    public MaxPQ(int maxN) {
        pq = (Key[])new Comparable[maxN+1];
    }
    public boolean isEmpty(){
        return N == 0;
    }
    public void insert(Key v){
        pq[++N] = v;
        swim(N);
    }
    public Key delMax(){
        Key max = pq[1];
        exch(1, N--);//将其和最后一个节点交换
        pq[N+1] = null;//移除最后一个元素
        //exch(1, N); pq[N--]=null;
        sink(1);//下沉
        return max;
    }
}
```
#### 堆的构造
> 简单的，从左至右遍历数组，用`swim()`保证扫描指针左侧的所有元素是一颗堆有序的完全树，就像连续向优先队列插入元素一样

高效的方式：从右至左用`sink()`函数构造子堆；
如果一个结点的两个子结点都已经是堆了，在该结点上调用`sink()`可以将它们变成一个堆
这个过程会`递归`建立堆的秩序,开始时只需要扫描数组中的`一半`元素，跳过大小为1的子堆，最后在位置1上调用`sink()`

> 堆中父节点有N/2个

```java
//见《算法》207
public static void sort(Comparable[] a) {
    int N = a.length;
    //得到堆有序
    for(int k = N/2; k >=1; k--) {
        sink(a, k, N);
    }
    //堆排序:将跟结点移到末尾，最终实现从小到大排列
    while(N >1){
        exch(a, 1, N--);//将顶端元素与末尾交换，并减小堆长度
        sink(a, 1, N);//做堆有序
    }
}
```
> 下沉排序与选择排序类似，区别是下沉排序提供了一种从未排序部分找到最大元素的有效方法；

`稳定性`:如果一个排序算法能够保留数组中重复元素的相对位置则可以称为是`稳定的`
稳定的: `插入排序`，`归并排序`
不稳定: `选择排序`，`希尔排序`，`快速排序`，`堆排序`

### 排序算法比较
| 算法 | 是否稳定 | 是否原地排序 | 时间复杂度 | 空间复杂度 | 备注 |
| --- | :-: | :-: | --- | :-: | :-: |
| 选择排序 | 否 | 是 | $N^2$ | 1 | 取决于输入元素的排列情况 |
| 插入排序 | 是 | 是 | 介于N与$N^2$之间 | 1 | 同上 |
| 希尔排序 | 否 | 是 | NlogN?$N^{6/5}$ | 1 | 同上 |
| 快速排序 | 否 | 是 | NlogN | lgN | 运行效率由概率提供保证 |
| 三向快速排序 | 否 | 是 | 介于N和NlogN之间 | lgN |  |
| 归并排序 | 是 | 否 | nlogN | N |  |
| 堆排序 | 否 | 是 | NlogN | 1 |  |



[^a]: 算法的稳定性
    保证排序前2个相等的数其在序列的前后位置顺序和排序后它们两个的前后位置顺序相同
    不稳定排序算法: 堆排序、快速排序、希尔排序、直接选择排序
    稳定的排序算法: 基数排序、冒泡排序、直接插入排序、折半插入排序、归并排序



