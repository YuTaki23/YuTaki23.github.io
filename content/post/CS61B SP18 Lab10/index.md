---
title: CS61B SP18 Lab 10 Priority Queues
description: 关于CS61B的第10个Lab的个人解法
date: 2025-04-06
categories:
    - CS61B
    - 解题思路
tags:
    - Java
    - 学习笔记
comment: true
---

## 具体思路
本lab大致内容为让我们使用数组来实现一颗树，且树要有类似于BST的搜索功能，但这里的比较方法不是简单的比较，而是每个节点都存在一个优先级，值越小优先级越大。题目给了我们许多脚手架，我的建议是一直看下去知道`size()`方法，连同注释也要一起看，这样就会对整个内容有个大致的了解，这很重要。
以下做题顺序以lab建议我们的做题顺序为准。
### `left/right/parentIndex(int i)`
这三个方法只需要用题目给我们的**representing a tree with an array**来做即可，具体为
- 节点n的左节点的位置位于**2n**
- 节点n的右节点的位置位于**2n+1**
- 节点n的父节点的位置位于**n/2**
可写出代码
```java
private static int leftIndex(int i) {  
    return 2 * i;  
}  

private static int rightIndex(int i) {  
    return 2 * i + 1;  
}  
  
private static int parentIndex(int i) {  
    return i / 2;  
}
```
### `swim(int index)`
先看一下题目对于**swim**的定义
1. 将刚才添加的项与其父项交换
2. 直到刚才添加的项比其父项大，否则一直交换值
这样一步一步向上游的感觉就称之为swim
问：这里的比较比的是什么？
答：各个数组元素为一个节点，每个节点存在一个优先级`mypriority`
使用上文给我们的方法`min`即可比较两节点之间的优先级大小，可以写出代码，
```java
private void swim(int index) {  
    // Throws an exception if index is invalid. DON'T CHANGE THIS LINE.  
    validateSinkSwimArg(index);  
    while (min(index, parentIndex(index)) == index &&  
            inBounds(parentIndex(index))) {  
        swap(index, parentIndex(index));  
        index = parentIndex(index);  
    }  
}
```
### `sink(int index)`
先看一下题目对**sink**的定义
将新节点逐渐sink下来，由于树的不变量，它必须要比其两个子树大才能保持原位，若比其两子树都小则要选择一个更小的，从更小的身上swap下来，直到比两子树大，故可以写出代码
```java
private void sink(int index) {  
    // Throws an exception if index is invalid. DON'T CHANGE THIS LINE.  
    validateSinkSwimArg(index);  
    while (true) {  
        int left = leftIndex(index);  
        int right = rightIndex(index);  
        int smallest = index;  
  
        if (min(left, smallest) == left) {  
            smallest = left;  
        }  

        if (min(right, smallest) == right  ) {  
            smallest = right;  
        }  
  
        if (smallest != index) {  
            swap(index, smallest);  
            index = smallest;  
        } else {  
            break;
        }  
    }  
}
```
### `insert(T item, double priority)`
看一下题目对add an item的要求
1. 将要添加的节点放在树最底部的位置，即数组最后面的位置
2. swim此节点
对于此树，数组0的位置没有任何东西，故不能简单的用size来表示当前树有多少个节点，而是用`size+1`，故写出代码
```java
public void insert(T item, double priority) {  
    /* If the array is totally full, resize. */  
    if (size + 1 == contents.length) {  
        resize(contents.length * 2);  
    }  
  
    Node currentNode = new Node(item, priority);  
    size += 1;  
    contents[size] = currentNode;  
    swim(size);  
}
```
### `peek()`
根据注释要求，简单返回第一个节点的item即可，不过在此还是考虑一下边界情况
```java
if (size <= 0) {  
    return null;  
} else {  
    return contents[1].myItem;  
}
```
### `removeMin()`
看一下题目给的定义
1. 将根节点与最右边（最小）的节点交换
2. 删除最右边的节点
3. sink新节点
但在这里还需要返回被删除的节点，故应该在最前面加上初始化，最后再return，简单写出代码
```java
public T removeMin() {  
    T res = peek();  
    swap(1, size);  
    contents[size] = null;  
    size--;  
    sink(1);  
    return res;  
}
```
[GitHub地址](https://github.com/YuTaki23/CS61B-SP18/tree/main/lab10)