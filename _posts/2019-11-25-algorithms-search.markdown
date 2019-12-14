---
layout: post
title:  "<算法>_查找"
date:   2019-11-25 09:28:02 +0800
categories: jekyll update
tags: algorithms java
---

##算法
[TOC]
### 顺序查找(基于无序链表)

```java
public class SequentialSearchST<Key, Value> {
    private Node first;//链表首结点
    private class Node {
        Key key;
        Value val;
        Node next;
        public Node(Key key, Value val, Node next){
            this.key = key;
            this.val = val;
            this.next= next;
        }
    }
    public Value get(key key){
        for (Node x = first; x!= null; x=x.next){
            if(key.equals(x.key){
                return x.val;
            }
        }
        return null;
    }
    public void put(Key key, Value val){
        for(Node x= first; x!= null; x = x.next){
            if(key.equals(x.key)) {
                x.val = val;
                return ;
            }
        }
        first = new Node(key, val, first);//未命中，新建结点
    }
}
```
### 二分查找(基于有序数组)

```java
public class BinarySearchST<Key extends Comparable<Key>,Value>{
    private Key[] keys;
    private Value[] vals;
    private int N;
    public BinarySearchST(int capacity){
        keys = (Key[]) new Comparable[capacity];
        vals = (Value[]) new Object[capacity];
    }
    public int size(){
        return N;
    }
    public Value get(Key key){
        if (isEmpty()) return null;
        int i = rank(key);
        if(i < N && keys[i].compareTo(key) == 0)
            return vals[i];
        else
            return null;
    }
    
    public void put(Key key, Value val){
        int i= rank(key);
        if(i < N && keys[i].compareTo(key) == 0){
            vals[i]= val;
            return ;
        }
        for(int j=N ; j>i; j--){
            keys[j] = keys[j-1];
            vals[j] = vals[j-1];
        }
        keys[i] = key;
        vals[i] = val;
        N++;
    }
}
```
```java
//二分查找
public int rank(Key key){
    int lo =0, hi = N-1;
    while(lo <= hi){
        int mid = lo+ (hi-lo)/2;
        int tmp= key.compareTo(keys[mid]);
        if (tmp <0)
            hi = mid -1;
        else if(tmp > 0)
            lo = mid +1;
        else
            return mid;
    }
    return 0;
}
```

### 符号表的各种实现的优缺点

| 使用的数据结构     | 实现                                                    | 优点                                               | 缺点                                                         |
| --- | --- | --- | --- |
| 链表(顺序查找)     | SequentialSearchST                                      | 适用小型问题                                       | 大型符号表查找慢                                             |
| 有序数组(二分查找) | BinarySearchST                                          | 最优的查找效率和空间需求，能够进行有序性相关的操作 | 插入操作慢                                                   |
| 二叉查找树         | BST                                                     | 实现简单，能进行有序性相关的操作                   | 没有性能上界的保证，链接需要额外的空间                       |
| 平衡二叉查找树     | RedBlackBST                                             | 最优的查找和插入效率，能进行有序性相关的操作       | 链接需要额外的空间                                           |
| 散列表             | SeparateChainHashST                 LinearProbingHashST | 能够快速的查找和插入常见类型的数据                 | 需要计算每种类型的数据的散列，无法进行有序性相关的操作，链接和空结点需要额外的空间 |

### 二叉查找树《算法》250

每个结点都含有一个Comparable的键且每个结点的键都大于其左子树中的任意结点的键，小于右子树的任意结点的键

```java
public class BST<Key extends Comparable<Key>, Value> {
    private Node root;
    private class Node {
        private Key key;
        private Value val;
        private Node left, right;
        private int N;//该结点为根的子树中的结点总数
        
        public Node(Key key, Value val, int N){
            this.key = key;
            this.val = val;
            this.N = N;
        }
    }
    public int size(){
        return size(root);
    }   
    private int size(Node x){
        if (x == null)
            return 0;
        else 
            return x.N;
    }
    
    public Value get(Key key){
        return get(root, key);
    }
    private Value get(Node x, Key key){
        if (x == null) return null;
        int tmp = key.compareTo(x.key);
        
        if (tmp < 0)
            return get(x.left, key);
        else if (tmp > 0)
            return get(x.right ,key);
        else
            return x.val;
    }
    
    public void put(Key key, Value val){
      //查找key，找到则更新它的值，否则为它创建一个新的结点
        root = put(root, key, val);
    }
    private Node put(Node x, Key key, Value val){
      //创建新结点
        if (x == null) return new Node(key, val, 1);
      
        int tmp = key.compareTo(x.key);
        if (tmp <0) 
            x.left = put(x.left, key ,val);
        else if(tmp >0)
            x.right = put(x.right, key, val);
        else
            x.val = val;
        
        x.N = size(x.left)+size(x.right)+1;
        return x;
    }
    //最小值
    public Key min() {
        return min(root).key;
    }   
    private Key min(Node x){
        if (x.left == null) return x;
        return min(x.left);
    }
    //小于等于key的最大键
    public Key floor(Key key) {
        Node x = floor(root, key);
        if (x == null) 
            return null;
        return x.key;
    }
    private Node floor(Node x, Key key) {
        if (x == null) return null;
        int cmp = key.compareTo(x.key);
        if (cmp == 0) return x;
        //x.key > key 向x左子树查找小于等于key的值
        if (cmp < 0) return floor(x.left, key);
        
        Node t = floor(x.right , key);
        if (t != null) return t;
        else return x;
    }
    
    //**rank()是select()的逆方法**
    
    public Key select(int k) {
        return select(root, k).key;
    }
    //返回排名为k的结点
    private Node select(Node x, int k) {
        if (x == null) return null;
        int t = size(x.left);
        if (t > k) return select(x.left, k);
        else if(t < k) return select(x.right, k-t-1);
        else return x;
    }
    
    public int rank(Key key) {
        return rank(key, root);
    }
    //返回x结点的子树中小于x.key的键的数量
    private int rank(Key key, Node x) {
        if (x == null) return 0;
        int cmp = key.compareTo(x.key);
        if (cmp < 0) return rank(key, x.left);
        else if( cmp > 0) return 1+size(x.left)+rank(key, x.right);
        else return size(x.left);
    }
    
    //删除最小值
    public void deleteMin(){
        root = deleteMin(root);
    }
    //递归遍历，深入跟结点的左子树直至遇到一个空链接，返回它的右子树即可
    private Node deleteMin(Node x) {
        if (x.left == null) return x.right;
        x.left = deleteMin(x.left);
        x.N = size(x.left) + size(x.right) + 1;
        return x;
    }
    
    //删除
    public void delete(Key key) {
        root = delete(root, key);
    }
  /**
  	*1.将指向即将被删除的结点的链接保存为t
  	*2.将x指向它的后继结点min(t.right)
  	*3.将x的右链接指向deleteMin(t.right),也就是在删除后所有结点仍然都大于x.key的子二叉查找树
  	*4.将x的左链接设为t.left
  	*/
  	private Node delete(Node x, Key key){
      if (x == null) return null;
      int cmp = key.compareTo(x.key);
      if (cmp < 0) x.left = delete(x.left, key);
      else if(cmp > 0) x.right = delete(x.right, key);
      else {
        if (x.right == null) return x.left;
        if (x.left == null) return x.right;
        Node t = x;
        x = min(t.right);
        x.right = deleteMin(t.right);
        x.left = t.left;
      }
      x.N = size(x.left)+size(x.right)+1;
      return x;
    }
  
  	//范围查找
	  public Iterable<Key> keys() {
      return keys(min(), max());
    }
  	public Iterable<Key> keys(Key lo, Key hi){
      Queue<Key> queue = new Queue<Key>();
      keys(root, queue, lo, hi);
      return queue;
    }
  	private void keys(Node x, Queue<Key> queue, Key lo, Key hi){
      if (x == null) return ;
      int cmplo = lo.compareTo(x.key);
      int cmphi = hi.compareTo(x.key);
      if (cmplo < 0 ) keys(x.left, queue,lo, hi);
      if (cmplo <=0 && cmphi >=0) queue.enqueue(x.key);
      if (cmphi > 0) keys(x.right, queue, lo ,hi);
    }
  	
    //大于等于key的最小键
    public Key ceiling(Key key){
        //TODO
    }
  	public Key max(){ //TODO }
}
```

### 平衡查找树
#### 2-3查找树
一颗2-3查找树由一下结点组成：
`2-结点`：含有一个键和两条链接，左链接指向的2-3树中的键都小于该结点，右链接指向的2-3树中的键都大于该结点；
`3-结点`：含有两个键和三条链接，左链接指向的2-3树中的键都小于该结点，中链接指向的2-3树中的键都位于该结点的两个键之间，右链接指向的2-3树中的键都大于该结点。

一颗`完美平衡`的2-3查找树中所有的空链接到跟结点的距离都应该是相同的

* **向2-结点中插入新键**
只要把2-结点替换为一个3-结点，将要插入的键保存在其中即可

* **向3-结点中插入新键：**
先临时将新键存入该结点中，使之成为一个4-结点
将4-结点转换为一颗由3个`2-结点`组成的2-3树,其中一个结点(根)含有中键，左链接为3个键中的最小值，右链接为三个键中的最大值。
既是3个结点的二叉查找树，也是完美平衡的2-3树.

* **向一个父结点为`2-结点`的3-结点中插入新键**  

1. 构建临时4-结点并将其分解；
2. 将中键移动到父结点2-结点中，使之成为`3-结点`；
3. 临时4-结点的左键，右键分别为父结点的中链接，右链接；并分解为2个`2-结点`；

* **向一个父结点为`3-结点`的3-结点中插入新键**
向上不断分解临时的4-节点并将中键插入更高层的父结点，知道遇到一个2-结点将它替换为一个不需要继续分解的3-结点，或到达3-结点的根

**分解根结点**
如果从插入结点到根结点的路径上都是3-结点，根结点最终变成`临时4-结点`，将4-结点分解为3个`2-结点`，使得树高加1 (按照`向3-结点中插入新键`处理)

> 插入新键原则：维持树的完美平衡，即**所有空链接到根结点的距离都应该是相同的**

2-3树插入算法的根本在于这些变换都是`局部的`,除了相关的结点和链接之外不必修改或检查树的其它部分
局部变换不影响树的全局有序性和平衡性

> * 变换之前后根结点到所有空链接的路径长度皆为h，  
> 当根结点被分解为3个2-结点时，所有空链接到根结点到路径长度会加1

*2-3树的生长是由下向上的*

#### 红黑二叉查找树(红黑树)
红链接将两个2-结点连接起来构成一个3-结点
黑连接则是2-3树中的普通链接
这种表示法的优点是直接使用标准二叉查找树的get()方法

红黑树另一种定义是含有红黑链接并满足下列条件的二叉查找树：

1. **红链接均为`左链接`；**
2. **没有任何一个结点同时和两条红链接相连；**
3. **该树是`完美黑色平衡`的，即任意空链接到根结点的路径上的黑链接数量相同**

将由红链接相连的结点合并，得到的就是2-3树
红黑树既是二叉查找树，也是2-3树；结合了二叉查找树中简洁高效的查找方法，2-3树中高效的平衡插入算法。

```java
private static final boolean RED = true;
private static final boolean BLACK = false;

private class Node {
    Key key;    //键
    Value val;  //值
  	Node left, right; //左右子树
    int N;      //该结点中子树的结点总数
    boolean color; //其父结点指向它的链接的颜色
    
    Node(Key key, Value val, int N, boolean color){
        this.key = key;
        this.val = val;
        this.N = N;
        this.color = color;
    } 
    
}
private boolean isRed(Node x){
    if (x == null) return false;
    return x.color == RED;
}
```

##### 旋转 (《算法》277)
某些操作可能会出现**红色右链接**或**两条连续的红链接**，使用旋转修复。使用旋转可以保持红黑树的两个重要性质：有序性和完美平衡性
`左旋转`：一条红色的右链接需要被转化为`左链接`
即将两个键中的较小者作为根结点变为较大者作为根结点

```java
Node rotateLeft(Node h){
    Node x = h.right;
    h.right = x.left;
    x.left = h;
    x.color = h.color;
    h.color = RED;
    x.N = h.N;
    h.N = 1+size(h.left)+size(h.right);
}

Node rotateRight(Node h){
    Node x = h.left;
    h.left = x.right;
    x.right = h;
    x.color =h.color;
    h.color = RED;
    x.N = h.N;
    h.N = 1+ size(h.left)+ size(h.right);
    return x;
}
```
##### 插入新键
* 向单个2-结点插入新键  
    如果新键 <老健，只需新增一个红色的结点，新的红黑树和单个3-结点完全等价
    如果新键 > 老健，新增的红色结点会产生一条红色右链接，使用`左旋转root=rotateLeft(root)`修正根结点的链接，插入完成
* 向数底部的2-结点插入新键 
    同单个2-结点
* 向一棵双键树(即一个3-结点)中插入新键  
    分三种情况：新键小于树中的两个键，在两者之间，大于树中两个键； 每种情况都会产生一个同时连接到**两条红链接**的结点,需要修正  
    1. 新键 > 树中的两个键；  
        新键被连接到`3-结点`的右链接，此时树平衡，根结点为中间大小的键，有两条红链接分别与较小和较大的结点相连；将两条链接的颜色**由红转黑**，得到一棵三个结点组成的高为2的平衡树. 是最简单情况
    2. 新键 < 树中两个键:  
        它被连接到最左边的空链接，此时产生两条连续的红链接； 将上层的红链接`右旋转`可得到第一种情况 (中值键为根结点并和其他两个结点用红链接相连)
    3. 新键介于树中的两个键之间:
        将下层的红链接`左旋转`可得到第二种情况

```java
//颜色转换： 局部操作，不影响整棵树的黑色平衡性
void flipColors(Node h){
    //父结点的颜色由黑变红
    h.color = RED;
    //将子节点的颜色由红变黑
    h.left.color = BLACK;
    h.right.color = BLACK;
}
```

**根结点总是黑色**
红色的根结点说明根结点是一个3-结点的一部分,每当根结点由红变黑时树的黑链接高度就加1

> 谨慎的使用`左旋转`，`右旋转`，`颜色转换`, 就能保证插入操作后`红黑树`和`2-3树`的一一对应关系  

**如果右子结点是红色而左子结点是黑色，进行`左旋转`；**
**左子结点是红色且它的左子结点也是红色，进行`右旋转`；**
**如果左右子节点均为红色，进行`颜色转换`;**


##### 红黑树的插入算法

```java
public class RedBlackBST<Key extends Comparable<key>, Value> {
    private Node root;
    
    private boolean isRed(Node h);
    private Node rotateLeft(Node h);
    private Node rotateRight(Node h);
    private void flipColors(Node h);
    private int size();
    
    //查找key，找到则更新值；否则新建一个结点
    public void put(Key key, Value val){
        root = put(root, key, val);
        root.color = BLACK;
    }
    private Node put(Node h, Key key, Value val){
    //标准的插入，和父结点用红链接相连
        if (h == null)
            return new Node(key ,val, 1, RED);
        int tmp = key.compareTo(h.key);
        if(tmp < 0)
            h.left = put(h.left, key, val);
        else if(tmp > 0)
            h.right = put(h.right, key, val);
        else
            h.val = val;
            
        //左旋转，右旋转，颜色转换
        if(isRed(h.right) && !isRed(h.left))
            h = rotateLeft(h);
        if(isRed(h.left) && isRed(h.left.left))
            h = rotateRight(h);
        if(isRed(h.left) && isRed(h.right))
            flipColors(h);
            
        h.N = size(h.left)+size(h.right) + 1;
        return h;
    }
}
```



结论: **所有基于红黑树的符号表实现都能保证操作的运行时间为对数级别**

