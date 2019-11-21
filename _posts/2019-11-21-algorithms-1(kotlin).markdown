---
layout: post
title:  "算法之排序(kotlin实现)"
date:   2019-11-21 17:15:02 +0800
categories: jekyll update
tags: [algorithms,kotlin]
---

```kotlin

fun main(){
//    val a = intArrayOf(20,5,10,3,7,98,2,9,54,1,23,6,65)
    val a = intArrayOf(20,5,10,3,7,98,2,9)
//    selection(a)
//    insertion(a)
//    shell(a)
//    mergeSort(a,0, a.size-1)
    quick(a, 0,a.size-1)
    a.forEach{
        print("$it,")
    }
}

/**
 * 快速排序
 */
fun quick(a:IntArray, start: Int, end: Int) {
    if (end <= start)
        return
    val partition = partition(a, start, end)
    quick(a, start, partition-1)
    quick(a, partition+1, end)
}
fun partition(a:IntArray, start: Int,end: Int): Int {
    var i = start //左右扫描指针
    var j = end+1
    val value = a[start]
    while (true){
        while (a[++i] < value)
            if (i == end)
                break
        while (value < a[--j])
            if (j == start)
                break
        if (i >= j)
            break
        swap(a, i, j)
    }
    swap(a, start, j) //将value放到索引j位置
    return j  //得到 a[start..j-1] <= a[j] <= a[j+1..end]
}

/**
 * 归并排序
 */
fun mergeSort(a: IntArray, start: Int, end: Int) {
    if (end <= start)
        return
    val mid = (start + end) / 2
    mergeSort(a,start, mid)
    mergeSort(a,mid+1, end)
    merge(a,start, mid, end) //
}

fun merge(a: IntArray, start:Int, mid:Int, end:Int){
    var i = start //左边起始索引
    var j = mid+1 //右边起始索引

    val tmp = IntArray(a.size)//create array & size = a.size
    a.forEachIndexed { index, i ->
        tmp[index] = i
    }

    for (k in start..end){
        when{
            i > mid -> { //左边越界
                a[k] = tmp[j++]
            }
            j > end -> { //右边越界
                a[k] = tmp[i++]
            }
            tmp[j] < tmp[i] -> {
                a[k] = tmp[j++]
            }
            else -> {
                a[k] = tmp[i++]
            }
        }
    }
}

/**
 * 希尔排序
 */
fun shell(a: IntArray) {
    val size = a.size
    var h = 1
    while (h<size/3)
        h = 3*h+1 //指定子数组个数
    while (h >=1 ) {
        for (i in h until size) {
            /**
             * 从h开始遍历，a[0..h]在遍历时会介入比较
             */
            for (j in i downTo h step h){
                if (a[j] < a[j-h])
                    swap(a, j, j-h)
            }
//            a.forEach {
//                print("$it ,")
//            }
//            println()
        }
        h /= 3 //递减h，当h=1，数组排序完成
    }
}

/**
 * 选择排序
 */
fun selection(a: IntArray) {
    val size = a.size

    for (i in 0 until size) {
        var min = i
        for (j in i + 1 until size) {
            if (a[j] < a[min])
                min = j
        }
        swap(a, min, i)
    }
}

/**
 * 插入排序
 */
fun insertion(a: IntArray) {
    val size = a.size
    //按升序排列
    for (i in 1 until size) { //不包含size
        /** i从索引1开始
         * 索引i与它之前的所有元素比较，
         * 如果i元素小，则与前面的元素交换索引；
         * 如果i大，则位置不变，
         * 直到j循环到索引0为止
         */
        for (j in i downTo 1) { //倒序遍历，包含1
            if (a[j] < a[j - 1]) {
                swap(a, j, j - 1)
            }
        }
    }
}

/**
 * 交换元素
 */
fun swap(a: IntArray, m: Int, n: Int) {
    val tmp = a[m]
    a[m] = a[n]
    a[n] = tmp
}

fun less(x: Int, y: Int) {

}
```

### 排序算法比较

| 算法 | 是否稳定 | 是否原地排序 | 时间复杂度 | 空间复杂度 | 备注 |
| --- | :-: | :-: | --- | :-: | :-: |
| 选择排序 | 否 | 是 | $N^2$ | 1 | 取决于输入元素的排列情况 |
| 插入排序 | 是 | 是 | 介于N与$N^2$之间 | 1 | 同上 |
| 希尔排序 | 否 | 是 | NlogN?$N^{6/5}$ | 1 | 同上 |
| 快速排序 | 否 | 是 | NlogN | lgN | 运行效率由概率提供保证 |
| 三向快速排序 | 否 | 是 | 介于N和NlogN之间 | lgN | - |
| 归并排序 | 是 | 否 | nlogN | N | - |
| 堆排序 | 否 | 是 | NlogN | 1 | - |

