---
title: CS106L Winter2025 Assignment5 Treebook
description: 关于CS106L的第五个作业的个人解法
date: 2025-04-17
categories:
    - CS106L
    - 解题思路
tags:
    - C++
    - 学习笔记
comment: true
---

## 写在前面
本文取自Stanford CS106L Winter2025 Assignment5: Treebook 本作业主要需要我们在其他人完成了一个类的情况下，将此类变得更加适用、更加好用，主要包括运算符重载、special member function等方面，总的来说不算难（比上次的好多了）
## 具体思路
### Viewing Profiles
目前，我们拥有一个User类，其中值得注意的是`_friends`这个field，它简单来说是一个vector，但更底层，是使用指针来实现的。
此题需要我们重载`<<`这个操作符，使得其打印时更符合我们人类的直觉
关于如何重载此操作符，如果没有思路，可以先看看这篇文章[为自己的类重载 `<<` 运算符](https://learn.microsoft.com/zh-cn/cpp/standard-library/overloading-the-output-operator-for-your-own-classes?view=msvc-170)
`User(name=Alice, friends=[Bob, Charlie])`，可以看出，除了简单的必须存在的字符串以外，我们还需要打印自己的`name`，friends的`name`，
自己的`name`很简单，类中已经给出方法`get_name()`
friends的`name`，就需要思考了，首先`friends`在private里，这意味着我们不能简单访问它们，而是使用**friend function**，其次还应明白如何访问一个指针元素，最后还要注意当我们到最后一个friend时，要改变打印的内容，因此可以写出代码
```cpp
std::ostream& operator<<(std::ostream& os, const User& us) {
  os << "User(name=" << us.get_name() << ", friends=[";

  for (size_t i = 0; i < us.size(); ++i) {
    if (i == us.size() - 1) {
      os << us._friends[i];
    } else {
      os << us._friends[i] << ", ";
    }
  }
  os << "])";
  return os;
}
```
### Unfriendly Behaviour
此题需要我们实现**Special Member Functions**，具体为
- Destructor
一般来说析构函数是不需要自己显式声明的，但若遇见了与**内存**相关的内容，就需要我们显式声明了，在此处，`_friends`就使用了内存，在全部完成后，需要释放内存
```cpp
  ~User();
```
```cpp
User::~User() {
  delete[] _friends;
}
```
- Copy Constructor
复制一个已存在的对象，将其声明为新的对象，值得注意的是，需要手动分配内存以保证`_friends`的正确性，在此处，笔者使用的是教授教的C++11的新方法，
```cpp
  User(const User& user);
```
```cpp
User::User(const User& user):
  _name(user._name),
  _size(user._size),
  _capacity(user._capacity),
  _friends(new std::string[user._capacity]) {
    for (size_t i = 0; i < user._size; ++i) {
      _friends[i] = user._friends[i];
    }
  }
```
- Copy Assignment
以上课时的内容为基准，举一反三既可
```cpp
  User& operator=(const User& user);
```
```cpp
User& User::operator=(const User& user) {
  if (this == &user) {
    return *this;
  }
  delete[] _friends;

  _name = user._name;
  _size = user._size;
  _capacity = user._capacity;
  _friends = new std::string[user._capacity];
  for (size_t i = 0; i < user._size; i++) {
    _friends[i] = user._friends[i];
  }
  return *this;
}
```
- Move Constructor & Move Assignment
此处要用到的内容为移动语义的知识，为了防止被移动，我们要显示删除这两个SMF，简单声明即可
```cpp
  User(User&& user) = delete;
  User& operator=(User&& user) = delete;
```
### Always Be Friending
此处需要我们重载两个操作符，以使得其更符合此类的操作
- `operator+=`
将一个用户添加到另一个用户上以成为其朋友，值得注意的是，**这是相互的，一个人对另一个人敞开心扉后，另一个人也应如此。**
最后还应按照题目要求返回`*this`
```cpp
  User& operator+=(User& rhs);
```
```cpp
User& User::operator+=(User& rhs) {
  this->add_friend(rhs.get_name());
  rhs.add_friend(this->get_name());
  return *this;
}
```
- `operator<`
简单判断第一个字符的先后即可
```cpp
  bool operator<(const User& rhs) const;
```
```cpp
bool User::operator<(const User& rhs) const {
  return this->get_name()[0] < rhs.get_name()[0];
}
```

[GitHub地址](https://github.com/YuTaki23/CS106L-Winter-2025/tree/main/assign5)