---
title: CS106L Winter2025 Assignment2 Marriage Pact
description: 关于CS106L的第二个作业的个人解法
date: 2025-04-09
categories:
    - CS106L
    - 解题思路
tags:
    - C++
    - 学习笔记
comment: true
---

## 写在前面
本作业取自Stanford CS106L Winter 2025 第2个Assignment。主要考察对读取文件内容、容器等内容的考察，总体难度不大，但是有几个坑，笔者会在下方标注**坑处**，如果你的测试过不去，大概率就是这里出了问题。
## 具体思路
### `kYourName`
将上方的名字改成自己的
```cpp
std::string kYourName = "YuTaki X";
```
### `get_applicants`
本题需要从一个`.txt`文件中读取其中内容，并将其中内容放到一个`set`里，具体思路即为
1. 打开文件
2. 读取每行的内容
3. 将此内容放到set中
4. 直到没有内容可读取
可以写出
```cpp
std::set<std::string> get_applicants(std::string filename) {
  std::set<std::string> students;
  std::ifstream inputFile;
  inputFile.open(filename);
  if (inputFile.is_open()) {
    std::string line;
    while (std::getline(inputFile, line)) {
      students.insert(line);
    }
  }
  inputFile.close();
  return students;
}
```
### `find_matches`
本体需要让我们找出与`name`有相同**initials**名字的人，并将其指针加入到一个queue中
> **坑点1**
> 什么是initials？
> 比如你的名字是Zhang San，那么initials就是ZS

具体思路为
1. 写一个辅助函数，用于得到每个名字的initials
2. 迭代查询set里的每个元素，若有与之相同的，则将其指针加入queue中
你需要知道的内容有
- 如何获取一个字符串中每个单词的第一个字母
> 坑点2
> 在用`stringstream`时，要在前面`include <sstream>`
- 如何迭代set
- 如何表示一个指针
```cpp
#include <sstream>
std::string helper(std::string name) {
  std::stringstream ss;
  ss << name;
  std::string first;
  std::string last;
  ss >> first >> last;
  std::string res;
  res += first[0];
  res += last[0];
  return res;
}

std::queue<const std::string*> find_matches(std::string name, std::set<std::string>& students) {
  std::queue<const std::string*> res;
  for (const auto& student : students) {
    if (helper(name) == helper(student)) {
      res.push(&student);
    }
  }
  return res;
}
```
### `get_match`
选择你自己喜欢的方法来从queue中再进一步得到一个特殊的名字（我的名字已经够特殊的了，就不再写了，其实是懒）
```cpp
std::string get_match(std::queue<const std::string*>& matches) {
  if (!matches.empty()) {
    return *matches.front();
  } else {
    return "NO MATCHES FOUND.";
  }
}
```
## 写在后面
总体不难，都是些小问题容易恼火
[GitHub地址](https://github.com/YuTaki23/CS106L-Winter-2025/tree/main/assign2)