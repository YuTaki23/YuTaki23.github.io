---
title: CS106L Winter2025 Assignment7 Unique Pointer
description: 关于CS106L的第七个作业的个人解法
date: 2025-04-19
categories:
    - CS106L
    - 解题思路
tags:
    - C++
    - 学习笔记
comment: true
---

## 写在前面

本文取自Stanford CS106L Winter2025 Assignment7: Unique Pointer

主要内容为运用模板、运算符重载、移动语义和智能指针。

本任务让我们自己写一个`unique_pointer`来实现链表的操作，感觉把后半部分的课程都串起来了，对Modern C++有了个更深的了解。

## 具体实现

### Implementing `unique_ptr`

本任务需要我们完成自己的`unique_ptr`，首先回忆一下唯一指针的内容。

唯一指针只能指向一个内容，无法让其他指针复制其内容，无法通过值传递，当变量超出范围时会调用delete，但在此题中无需担心delete。

#### Implementing `unique_ptr` functionality

此处需要我们完成几个特定的SMF

- `private`

这里需要我们在私有处完成内容，对于一个指针来说，它一定会存在的内容是指向一个内容的指针

```cpp
private:  
  T* ptr;
```

- `unique_ptr(T* ptr)`

此处完成一个构造函数，给出的参数为一个指针，让我们的唯一指针指向这个地方即可

```cpp
  unique_ptr(T* ptr) {
    this->ptr = ptr;
  }
```

- `unique_ptr(std::nullptr_t)`

此处完成一个指向`NULL`的构造函数，让唯一指针指向`NULL`即可

```cpp
  unique_ptr(std::nullptr_t) {
    this->ptr = NULL;
  }
```

- `T& operator*()`

此处需要重载`*`操作符，对于一个指针来说，`*`是解引用，返回的是指针指向的值，不是地址，所以不能使用`&`取地址符，简单的返回指针即可

```cpp
  T& operator*() {
    return *this->ptr;
  }
```

- `const T& operator*() const`

```cpp
  const T& operator*() const {
    return *this->ptr;
  }
```

- `const T* operator->() const`

此处需要返回唯一指针指向的内容

```cpp
  T* operator->() {
    return this->ptr;
  }
```

- `const T* operator->() const`

```cpp
  const T* operator->() const {
    return this->ptr;
  }
```

- `operator bool() const`

注释告诉我们若唯一指针为非NULL则返回true，反之相反

```cpp
  operator bool() const {
    return this->ptr != NULL; 
  }
```

#### Implementing RAII

此任务要求我们实现一些列SMF，以防止唯一指针不指向唯一的内容，同时保证不会进行复制等耗费内存的操作

- `~unique_ptr()`

析构函数，释放指针的内存，简单删除即可

```cpp
  ~unique_ptr() {
    delete ptr;
  }
```

- `unique_ptr(const unique_ptr& other)`

因为这是唯一指针，所以不能被其他指针复制，在此应该删除

```cpp
  unique_ptr(const unique_ptr& other) = delete;
```

- `unique_ptr& operator=(const unique_ptr& other)`

应删除，理由同上

```cpp
  unique_ptr& operator=(const unique_ptr& other) = delete;
```

- `unique_ptr(unique_ptr&& other)`

此处需要我们将`other`的指针移动到本指针上，同时因为这是唯一指针，不能存在两个指向相同地方的指针，给`other`的指针应指向`NULL`

```cpp
  unique_ptr(unique_ptr&& other) {
    this->ptr = other.ptr;
    other.ptr = NULL;
  }
```

- `unique_ptr& operator=(unique_ptr&& other)`

对`=`操作符重载，也就是将`this`赋值给`other`

```cpp
  unique_ptr& operator=(unique_ptr&& other) {
    if (this != &other) {
      delete this->ptr;
      this->ptr = other.ptr;
      other.ptr = NULL;
    }
    return *this;
  }
```

### Using `unique_ptr`

此部分需要我们使用刚才创建的唯一指针实现链表的操作，底层内容可以看一下README，在此不赘述

整体实现的算法题目也已经告诉我们了，一步一步跟着来即可，

值得注意的是，重新赋值时不能简单使用`=`来操作，而应该使用移动语义，

取vector的内容时，不能简单使用`[]`来操作，而应该使用唯一指针的操作

```cpp
template <typename T> cs106l::unique_ptr<ListNode<T>> create_list(const std::vector<T>& values) {
  cs106l::unique_ptr<ListNode<T>> head = nullptr;
  for (int i = values.size() - 1; i >= 0; --i) {
    cs106l::unique_ptr<ListNode<T>> node = cs106l::make_unique<ListNode<T>> (values[i]);
    node->next = std::move(head);
    head = std::move(node);
  }
  return head;
}
```

[GitHub地址](https://github.com/YuTaki23/CS106L-Winter-2025/tree/main/assign7)

## 写在后面

CS106L到此结束，简单了解了C++的特性，为下一门课CS144做准备。

这是笔者第一个把所有内容都写下来的一门课，虽一笔一划写下，但仍有许多不精之处，若您有更好的建议，请提出。

谢谢！

<p align = "right"> YuTaki </p>

<p align = "right"> 2025年4月19日，博学楼</p>