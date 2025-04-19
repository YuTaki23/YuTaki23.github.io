---
title: CS106L Winter2025 Assignment5 Explore Courses
description: 关于CS106L的第六个作业的个人解法
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
本文取自Stanford CS106L Assignment6: Explore Courses

本作业主要让我们使用`std::optional`这个模板来完善寻找Course这个方法，主要训练了我们对此模板的理解，总的来说，题目把大部分的提示都给我们了

## 具体思路

### Include `<optional>`

简单包含头文件即可

```cpp
#include <optional>
```

### Write the `find_course` function

此函数需要我们接受一个String类型的course_title，并返回courses中与其同名的course，整个函数逻辑很简单，简单的for循环遍历即可。

但难点在于还需要更改函数类型，关于这个可以先往下看，对于整个文件来说我们都是在基于`optional`上操作的，在之后还需要运用此函数返回的对象来进行操作，而在后文提到，这个对象首先得有`optional`的接口，故可以发现应将其写成以`optional`为基础的类型

题目还告诉我们，有可能没有相符的course，也就是没有遍历到合适的course，而在`optional`中，表示未发现的对象应使用`null_opt`

```cpp
  std::optional<Course> find_course(std::string course_title)
  {
    for (const auto& course : courses) {
      if (course.title == course_title) {
        return course;
      }
    }
    return std::nullopt;
  }
```

### Modifying the `main` function

题目首先告诉我们要调用上文的函数，上文已经解释过，在此不赘述

题目告诉我们要将下列行为以`monadic`操作符进行重写，且不使用任何条件函数

```cpp
if (course.has_value()) {
    std::cout << "Found course: " << course->title << ","
            << course->number_of_units << "," << course->quarter << "\n";
} else {
    std::cout << "Course not found.\n";
}
```

此函数要表达的意思很简单，若存在一个值打印一些内容，反之打印另一些内容

下面给了我们三个函数，在此笔者简单解释一下

- `and_then()`

若当前optional有值，则调用括号内的函数返回新的optional，否则返回空

- `transform()`

若当前optioanl有值，则调用括号内的函数并将结果包装成optional，否则返回空

- `or_else()`

若当前optional有值，则返回它，否则调用函数生成新的optional

**如果vscode一直报错说找不到函数，在vscode的设置文件把c++标准改成23即可**

再看一下题目给我们的示例，要在函数内容调用一个lambda函数，

对于存在值的情况，使用`transform()`，不存在的情况使用`or_else()`

```cpp
    std::string output = course
      .transform([](Course course) { return "Found course: " + course.title + "," + course.number_of_units + "," + course.quarter + "\n";})
      .or_else([]() { return std::optional<std::string>("Course not found.\n");})
      .value();
```

[GitHub地址](https://github.com/YuTaki23/CS106L-Winter-2025/tree/main/assign6)
