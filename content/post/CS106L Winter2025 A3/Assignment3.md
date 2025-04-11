---
title: CS106L Winter2025 Assignment3 Make a Class
description: 关于CS106L的第三个作业的个人解法
date: 2025-04-11
categories:
    - CS106L
    - 解题思路
tags:
    - C++
    - 学习笔记
comment: true
---

## 遇到的问题
最后的编译过不去，不知道为什么卡在
`Installing CastXML`就不动了
：（
## 具体思路
自由发挥创造一个类，没什么好解释的，创造性很强，我没有更好的想法，就用了上课用的`StanfordID`类来做
`class.h`
```cpp
#include <string>
#include <sstream>

class StanfordID {
    private:
        std::string name;
        std::string sunet;
        int idNumber;

        std::string getInitialName(std::string name);
    
    public:
        StanfordID(std::string name, std::string sunet, int idNumber);
        StanfordID();

        const std::string getName();
        const std::string getSunet();
        const int getIdNumber();

        void setName(std::string name);
};
```
`class.cpp`
```cpp
#include "class.h"
#include <string>

StanfordID::StanfordID(std::string name, std::string sunet, int idNumber) {
    this->name = name;
    this->sunet = sunet;
    if (this->idNumber > 0) {
        this->idNumber = idNumber;
    }
}

std::string StanfordID::getInitialName(std::string name) {
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

StanfordID::StanfordID() {
    this->name = "YuTaki X";
    this->sunet = "172@60";
    this->idNumber = 190;
}

const std::string StanfordID::getName() {
    return this->name;
}

const std::string StanfordID::getSunet() {
    return this->sunet;
}

const int StanfordID::getIdNumber() {
    return this->idNumber;
}

void StanfordID::setName(std::string name) {
    this->name = name;
}
```
`sandbox.cpp`
```cpp
#include "class.h"
void sandbox() {
  StanfordID id("Yammy Xi", "183@80", 187);
}
```