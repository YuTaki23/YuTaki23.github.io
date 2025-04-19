---
title: CS106L Winter2025 NOTES
description: CS106L的课程笔记
date: 2025-04-06
categories:
    - CS106L
    - 笔记
tags:
    - C++
    - 学习笔记
comment: true
---

## 1. Types and Structs
### 1.1 什么是C++？
1. 如何执行C++代码
C++是一个编译性语言，不同于Python的解释性语言，目前大部分C++程序所用的编译器为**g++**，作为编译性语言，C++拥有以下优缺点
- 允许生成更有效的机器语言，因为解释器只能看到代码的很小的一部分，但是编译器可以看到所有事物。
- 编译需要的时间很长。
2. 如何检查C++代码
在执行C++代码时，如果遇到错误，大多数情况会遇到很多十分冗余的日志，这让人很烦躁，为什么？
- **Types** C++是一个静态类型语言，每个变量都有自己的类型，一旦此变量被声明就无法再改变。虽然这不如Python方便，但也有许多好处，更有效、更好理解、更有效地错误检查。当然也会遇见*强制转换*的时候，`(int) 5.3 = 5`。C++支持**重载**操作，对于不同类型的值，只要声明两个函数包括两个不同的类型参数，C++会自己寻找该使用哪个函数。
综上，C++是一个**编译性、静态类型**的语言。
### 1.2 Structs
#### 1.2.1 结构体基础知识
当我们想要在一个类型里面表示许多不同的值，此时就需要用到结构体，例如，
```cpp
// 构造结构体
struct StanfordID {
	string name;
	string sunet;
	int idNumber;
}
```
在C++中，初始化一个结构体有许多方法
1. **初始化结构体** 使用点表达式
```cpp
StanfordID id;
id.name = "Alice";
id.sunet = "jtrb";
id.idNumber = 6504417;
```
2. **List Initialization** 使用大括号
```cpp
StanfordID jrb = {"Jacob Roberts-Baca", "jtrb", 6504417};
StanfordID fi {"Fabio Ibanez", "fibanez", 6504418} //此处的“=”可以省略
```
当我们想要表示*两个值*作为一个结构体时，`STL`已经给了我们模板，即`std::pair`
```cpp
// 结构体
struct Order {
	std::string item;
	int quantity;
}
Order dozen = {"Eggs", 12};

// std::pair
std::pair<std::string, int> dozen {"Eggs", 12};
std::string item = dozen.first;
int quantity = dozen.second;
```
可以看出，`pair`用的是`first`和`second`来表示两个值的。
#### 1.2.2 什么是`std`
`std` C++ Standard Libary
- 内置的类型、函数......
- 需要在文件前写`#include`才可以使用，本质上就是引用了其他包内的函数等
- 在使用`std`时，都会写上`std::`，但若在文件前写`using namespace std`就不用再写了
### 1.3 改善代码的一些方法
#### 1.3.1 `using`
在遇到一些类型十分复杂的函数时，其往往会存在大量的、不好理解的类型，此时就可以使用`using`关键字，例如
```cpp
// 使用 using 前
std::pair<bool, std::pair<double, double>> solveQuadratic(double a, double b, double c);

// 使用 using 后
using Zeros = std::pair<double, double>;
using Solution = std::pair<bool, Zeros>;
Solution solveQuadratic(double a, double b, double c);
```
#### 1.3.2 `auto`
场景如上，有时我们自己也搞不清这个变量到底是什么类型，那么此时就可以使用`auto`关键字，让编译器自己判断这个变量到底是什么类型
```cpp
// 使用 auto 前
std::pair<bool, std::pair<double, double>> result = solveQuadratic(a, b, c);

// 使用 auto 后
auto result = solveQuadratic(a, b, c);
```
这同样也满足静态类型的要求，不会将一个int类型的值赋给String
## 2. Initialization & References
### 2.1 Initialization
在初始化一个值时，C++有许多种方法来实现
#### 2.1.1 Direct initialization
```cpp
int numOne = 12.0;
int numTwo(12.0);
```
在**直接初始化**中，C++不在乎你的值是否满足类型检查，它只会简单地将其看作你给他赋予的类型
#### 2.1.2 Uniform initialization(C++11)
对于统一初始化来说，C++在乎你的值满足类型检查，简单说，若你给的值是个浮点数，同时给的类型是`int`，那么此时编译器变不会编译
```cpp
int numOne{12};
float numTwo{12.0};

int numThree{12.0} //不会编译
```
**优点**
1. 安全性 不允许发生*宅化转换*，即类型的精度变小，这避免了很多问题
2. 泛用性 几乎所有类型都可以用这个方法来初始化，例如`vector` `map` `class`
```cpp
// map
std::map<std::string, int> ages{
	{"Alice", 25},
	{"Bob", 30},
	{"Charlie", 35}
}

// vector
std::vector<int> numbers{1, 2, 3, 4, 5};
```
#### 2.1.3 Structured Bingding(C++17)
- 在编译时，想要初始化一些具有固定的大小且此变量是作为一个数据结构存在时就使用**结构化绑定**
- 拥有从一个函数的返回值一次性访问多个值的能力
比如，我们现在创建一个函数`getClassInfo`，它是用`tuple`这个数据结构来实现的，返回值使用的是**统一初始化**
```cpp
std::tuple<std::string, std::string, std::string> getClassInfo() {
	std::string className = "CS106L";
	std::string buildingName = "Thornton 110";
	std::string language = "C++";
	return {className, buildingName, language};
}
```
那么现在我就可以把这个函数返回的三个值用结构化绑定的方法来返回了
```cpp
int main() {
	auto [className, buildingName, lanuage] = getClassInfo();
}
```
不仅如此，我还可以使用`get`来获取函数的返回值，值得注意的是，在这里需要使用尖括号来寻找第几个元素
```cpp
int main() {
	auto classInfo = getClassInfo();
	std::string className = std::get<0>(classInfo);
	std::string buildingName = std::get<1>(classInfo);
	std::string language = std::get<2>(classInfo);
}
```
- 可以在编译时对已知大小的对象上使用
### 2.2 References
####  2.2.1 References简单介绍
- 什么**References**？将一个已经存在的对象或函数的别名。
- 如何表示**References**？使用`&`。
例如
```cpp
int num = 5;
int& ref = num;
```
num是一个整数类型变量，值为5；ref是一个整数引用类型变量，值为num，即num的别名。当我们更改ref的值时，也是在更改num的值。即ref是指向num的一个指针，由此我们就可以引出*pass by reference*
#### 2.2.2 Pass by reference
当我们在函数中声明参数为引用类型时，即代表此函数使用的是*pass by reference*，这也就意味着，我们不会在内存中再分配一块内存来复制此变量等执行一系列操作，只会在原内存上操作
```cpp
void squareN(int& n) {
	n = std::pow(n, 2);
}

int main() {
	int num = 2;
	squareN(num);
}
```
在这里如果我们再去打印`num`的值，得到的结果就会是**4**，而不是**2**
#### 2.2.3 Pass by value
这意味着我们会在内存中重新分配一块内存，并将原参数复制一份过来，不会在原内存上进行操作，不会影响原内存。
```cpp
void squareN(int n) {
	n = std::pow(n, 2);
}

int main() {
	int num = 2;
	squareN(num);
}
```
在这里再去打印num的值得到的结果就是**2**
#### 2.2.4 按值传递与按引用传递易错点
值得注意的是，当我们在遇见一些很复杂的类型时，会用到之前学到的**auto**，同时还用了**Structred Binding**但在进行按引用传递时，编译器不会帮我们寻找 **&** 我们需要自己加上 **&** 即
```cpp
void shift(std::vector<std::pair<int, int>> &nums) {
	for (auto& [num1, num2] : nums) {
		num1++;
		num2++;
	}
}
```
但当我们用普通的**点表达式**时，就不需要这样，
```cpp
void shift(std::vector<std::pair<int, int>> &nums) {
	for (size_t int i = 0; i < nums.size(); i++) {
		nums[i].first++;
		nums[i].second++;
	}
}
```
### 2.3 L-values vs R-values
左值和右值是表达式的基本分类，用于描述其值类别，区别在于对象的身份、生命周期和可操纵性。
左值代表一个有持久状态的变量（或内存位置），通常可以取地址、有名称。可以出现在表达式的左边或右边。
右值代表一个临时变量或字面量，没有持久状态，不可以被取地址。出现在表达式的右边。
```cpp
int x = 10; // x为左值，10为右值
int y = x; // x为左值
```
对于`int& num`是左值还是右值？
左值。在这里，num是按引用传递，对于右值来说只可以存放临时变量，不可以被取地址，所以这里的num只能是左值。
### 2.4 Const
当一个对象以常数声明后，它就无法再被修改了。
比如，对于一个vector，若被声明为`const std::vector<int> const_vec;`，那么此时若对此vector使用`push_back`就会编译错误。
若将一个引用非常数类型的引用一个常数类型的vector则会导致编译错误
```cpp
const std::vector<int> const_vec{1, 2, 3};
std::vector<int>& bad_ref{const_vec}; // 编译错误
const std::vector<int> good_ref{const_vec}; //编译成功
```
### 2.5 Compiling C++ Programs
- C++是一个编译性语言
- 存在一款软件——编译器
- 目前流行的编译器有**clang** **g++**
`g++ -std=c++20 main.cpp -o main`
此命令的内容为 使用g++这个编译器在C++20的环境下编译main.cpp这个文件并将编译后的程序命名为main
最后使用`./main`来执行此程序。
## 3. Streams
### 3.1 Streams简单介绍
C++的一个普遍的Input/Output（IO）抽象，此抽象提供一系列接口，用于读取和写入数据。
```cpp
std::cout << "Hello, World" << std::endl;
```
`std::cout`是一个`std::ostream`的一个实例变量，用于打印内容
```cpp
double pi;
std::cin >> pi;
```
`std::cin`是一个`std::istream`的一个实例变量，用于读取内容
流允许以一种普遍的方式来解决数据问题
### 3.2 stringstreams
#### 3.2.1 stringstreams简单介绍
用于将字符串以流的方式表达
对于不同类型数据混合的情况很适用
```cpp
std::string initial_quote = "Bjarne Stroustrup C makes it easy to shoot yourself in the foot"
std::stringstream ss;
ss << initial_quote;
```
将上文字符串以流的形式写入`ss`，并且最后存在`\n`
```cpp
std::string first;
std::string last;
std::string language, extracted_quote;
ss >> first >> last >> language;
```
对于stringstreams来说，以空格来分割一个字符串即，此时`first = "Bjarne"` `last = Stoustrup` `language = C`
但此时我们想把最后剩余的其他内容都写入`extracted_quote`用这种方式就不可取，由此引入`getline()`
#### 3.2.2 getline()
`getline()`会读取一行的内容，即以`\n`截止
```cpp
std::getline(ss, extracted_quote);
```
### 3.3 cout and cin
#### 3.3.1 cout的背后原理
当我们想将一个内容输出到终端或控制台时，应当使用`std::cout`和`<<`操作符，但仅这样没法输出到控制台，我们还需要加上`flush`
```cpp
double tao = 6.28;
std::cout << tao;
std::cout << std::flush;
```
在调用`endl`会隐式调用`flush`
```cpp
double tao = 6.28;
std::cout << tao;
std::cout << std::endl;
```
当我们每次调用`flush`时，就会立马清空此处的内存，包括使用`endl`的时候，在做一个循环时，当我们不断打印一个内容再加上一个换行符，就会耗费许多内存，故`flush`操作是很昂贵的。
但是，C++作为一个聪明的语言，是会自己知道什么时候该`flush`的，每当此处内存满了之后，就会自动`flush`
### 3.4 Output Streams
当我们想将一些内容写入文件中就需要用到`std::ofstream`
```cpp
int main() {
	std::ofstream ofs("hello.txt"); // 创建一个流用于打开文件
	if (ofs.is_open()) {
		ofs << "Hello CS106L" << '\n';
	} // 检查文件是否打开
	ofs.close(); // 关闭文件
	ofs << "this will not get written"; //此内容不会写进
	ofs.open("hello.txt"); //打开文件
	ofs << "this will though! It's open again"; // 此内容会写进文件
	return 0;
}
```
### 3.5 Input Streams
从一个文件读取内容就需要用到`std::istream`
`std::cin`会在遇见空格后停止
- `getline()`和`std::cin()`不能在一起使用
## 4. Containers
### 4.1 What is the STL?
**STL: Standard Template Library**
#### 4.1.1 What are templates?
我们总会遇见同一种函数，十分相似，但它们的差别就在类型不一样，例如
```cpp
class IntVecor
class DoubleVector
class StringVector
```
如果我们有几百上千种类型呢？难道要一个一个写出来吗，当然不，由此引出**模板**
```cpp
template <typename T>
class vector {

};

vector<int> v1;
vector<double> v2;
vector<string> v3;
```
**所有的STL容器都是模板**
#### 4.1.2 Containers
如何保存一系列元素
##### 4.1.2.1 Sequence Containers
序列式容器用于保存一些列线性元素
- **std::vector**
```cpp
std::vector<int> v; // 创建一个空vector
std::vector<int> v(n); // 创建一个大小为n的vector，其中所有元素均为0
std::vector<int> v(n, k); // 创建一个大小为n的vector，其中所有元素均为k
v.push_back(k); // 将k添加到vector后面
v.clear(); // 清空vector
v.empty(); // 检查vector是否为空
// 获取第i个元素
int k = v.at(i);
int k = v[i];
// 将第i个元素替换为k
v.at(i) = k;
v[i] = k;
```
Tips
1. 如果可以，使用**range-based for**（基本范围的for循环）
```cpp
for (size_t i = 0; i < vec.size(); i++) {
	std::cout << vec[i] << " ";
}
// range-based for
for (auto elem : vec) {
	std::cout << elem << " ";
}
```
这适用于所有可迭代的容器，不止vector
2. 在使用**range-based for**时，将类型更改为**const auto&**，这有利于节省内存，原先会创建一系列元素的复制，现在只会在原元素上更改
```cpp
for (auto elem : vec) 
变为
for (const auto& elem : vec)
```
3. 对于[]操作符不会进行边界检查
```cpp
// 假定2已经超出边界
vec[2] = 4; // 未定义的行为
vec.at(2) = 4; // 运行时间超时
```
假定现在又超过一百万个元素在一个vector中，想要在第一个地方添加一个元素，就需要将这一百万个元素全部向后移动一位，这很耗费时间，由此我们就引出了**deque**
- **std::deque**
deque是双端队列，允许操作第一个或最后一个元素
对于vector来说，用的是空间中的一组内存，而对于deque，用的书空间中一组内存的一组内存，将整个元素分开了
##### 4.1.2.2 Associative Containers
关联式容器用于组织键值对元素
- **std::map**
用一个key来匹配一个value，其中key是唯一的，value可以不唯一
```cpp
std::map<char, int> m; // 创建一个空map
// 将key为k，value为v的键值对添加到map中
m.insert({k, v});
m[k] = v;
m.erase(k); // 移除key为k的键值对
// 检查k是否在map里
m.count(k)
m.contains(k) // C++20
m.empty() // 检查map是否为空
// 将key为k的value重写
int i = m[k];
m[k] = i;
```
当map存储的是一个键值对时，其也可以被看作为pair
```cpp
std::map<K, V>
std::pair<const K, V>
```
此处K为常量，即K是不可以被修改的，在map中key也是不可以被修改的，所以此处不能变
**range-based for**
```cpp
std::map<std::string, int> map;
for (auto kv : map) {
	// 此处就是把map当作pair来看，用的是first和second
	std::string key = kv.first;
	int value = kv.second;
}

// 此处map就是map
for (const auto& [key, value] : map) {

}
```
map的底层原理即为**红黑树**，这也就意味着key必须要有**操作符<**，比如`std::map<std::ifstream, int> map`在这里就是错误的，因为ifstream无法比较。
- **std::set**
set存储一组**唯一**的元素
```cpp
std::set<char> s; // 创建一个空set
s.insert(k); // 将k添加到set中
s.erase(k); // 移除set中的k
// 检查k是否在set里
s.count(k);
s.contains(k) // C++20
s.empty() // 检查set是否为空
```
set是没有value的map
set的底层原理也是红黑树，这同样意味着需要存在**操作符<**
- **std::unordered_map** 和 **std::unordered_set**
此容器与上文提到的没有什么差别，不同的是底层实现为**哈希表**，通过哈希函数，将其转变为无序的
为什么使用无序？更平均的负载因子，当负载因子过高时，允许rehash
1. 通常来说，`unordered_map`要快于`map`
2. 但`unordered_map`要用更多的内存
3. 若`key`不存在**操作符<**，使用`unordered_map`
4. 若必须要从中挑一个，`unordered_map`会更安全
## 5. Iterators
### 5.1 Iterator基础
在上文提到的range-based for中，这是如何实现的
容器与迭代器一起工作就构成了迭代
迭代器存在三个操作
- 获取一个元素
- 向前移动一个元素
- 检查整个元素是否已经遍历完成
容器存在两个接口
- `container.begin()`获取*第一个元素*迭代器
- `container.end()`获取*最后一个元素的后一个元素*的迭代器，它永远不会指向最后一个元素，一定会指向最后一个元素的后一个元素，因为若此容器为空，`c.begin() == c.end()`就会为真
综上，当我们写下
```cpp
for (auto elem : s) {
	std::cout << elem;
}
```
就是写下
```cpp
for (auto it = s.begin(); it != s.end(); ++it) {
	auto elem = *it;
	std::cout << elem;
}
```
补充：为什么使用`++it`而不是`it++`
`++it`会避免生成不必要的复制
### 5.2 Iterator Types
对于迭代器来说，不止有一个一个增加的迭代，还存在许多其他操作
```cpp
--it; // 向前一位
*it = elem; // 修改此处的值
it += n; // 向前n位
it1 < it2; // 判断it1是否在it2前面
```
若元素在结构体当中，可以用`->`来访问元素
```cpp
struct Bibble {
	int zarf;
};

std::vector<Bibble> v {...};
auto it = v.begin();
int m = (*it).zarf;
int m = it->zarf;
```
```cpp
// IO迭代器
auto elem = *it // 访问元素
*it = elem; // 写入元素

// 向前迭代器
it1 == it2；
++it1 == ++it2；

// 反向迭代器
auto it = m.end();
--it;
auto& elem = *it; // 获取最后一个元素

//随机访问迭代器
auto it2 = it + 5; // 向前五个
auto it3 = it2 - 2; // 向后两个
```
### 5.3 Pointers and Memory
一个**迭代器**指向一个**容器元素**
一个**指针**指向任何**对象**
#### 5.3.1 Memory基础
- 每个变量都存放在内存中的某个地方
- 所有地方都可以被一个地址来表示
- 地址用字节寻址法，每个字节都从0开始
- 1 byte = 8 bits
- 一个对象的地址是用此对象最小地址来表示的
对于一个整数类型x来说，它在内存中占四个字节，假设为0x10 0x11 0x12 0x13，那么此时x的地址就是0x10
#### 5.3.2 变量地址表示方法
**指针**
```cpp
int x = 106;
int* px = &x; // int*为一个整数指针 &为取地址操作符

x // 106
*px // 106
px // 0x50527c
```
对于不同的对象，有不同的指针表示方法
```cpp
int x = 106;
int* px = &x;

// 结构体
StanfordID id {"jtrb"};
StanfordID* p = &id;
auto name = p -> name;

// vector
std::vector<int> v;
std::vector<int>* p = &v;

// vector []表示法
std::vector<int> v {1, 2, 3, 4, 5};
int* arr = &v[0];
```
对于vector来说，其在内存中是以一块内存表示的，故我们可以使用**指针运算**来表示
```cpp
std::vector<int> v {1, 2, 3, 4, 5};
int* arr = &v[0];
arr += 1; // index = 1
++arr; // index = 2
arr += 2; // index = 4
arr == &v[4]; // 取index为4的值
```
## 6. Classes
### 6.1 类基础
1. 为什么使用类？
	- C没有对象
	- 不能在操作数据的时候封装数据与函数
	- 无法使用面向对象编程范式
2. 什么是面向对象
	- 面向对象所有内容都是围绕对象来实现的
	- 专注于设计与实现类
	- 类是由用户定义的
**容器都是STL定义的类**
### 6.2 结构体与类的区别
**结构体是受限制的类**
对于结构体来说，其所定义的所有`fields`都是`public`的，这也就意味着它们可以被用户随意更改
对于类，其存在`public` `private`，用户只能访问`public`的对象
### 6.3 类语法
#### 6.3.1 头文件与源文件的区别

|      | Header File(.h)        | Source File(.cpp)        |
| ---- | ---------------------- | ------------------------ |
| 目的   | 定义接口                   | 实现函数                     |
| 包括内容 | 函数原型、类声明、类型定义、宏、常数     | 函数具体实现内容，执行代码            |
| 访问   | 与源文件分享                 | 被编译                      |
| 举例   | `void someFunction();` | `void someFunction(){};` |
#### 6.3.2 设计类
- 构造函数
- 私有函数/变量
- 公共函数
- Destructor（析构函数）
```cpp
// .h file
class StanfordID {
private:
	std::string name;
	std::string sunet;
	int idNumber;

public:
	StanfordID(std::string name, std::string sunet, int idNumber); // Constructor
	// method
	std::string getName();
	std::string getSunet();
	int getID();
}
```
```cpp
// .cpp file
#include "StanfordID.h" // 包含头文件
#include <string>

// 使用namespace就像std::一样
StanfordID::StanfordID(std::string name, std::string sunet, int idNumber) {
	this -> name = name; // 使用this关键字来表示现在具体是哪个值
	this -> sunet = sunet;
	// 可以检查是否有效
	if (this -> idNumber > 0) {
		this -> idNumber = idNumber;
	}
}

// uniform initialization(C++11)
StanfordID::StanfordID(std::string name, std::string sunet, int idNumber) : name{name}, sunet{sunet}, idNumber{idNumber};

// 实现方法
std::string StanfordID::getName() {
	return this->name;
}
std::string StanfordID::getSunet() {
	return this->sunet;
}
int StanfordID::getID() {
	return this->idNumber;
}

// 绑定函数
void StanfordID::setName(std::string name) {
	this->name = name;
}
void StanfordID::setSunet(std::string sunet) {
	this->sunet = sunet;
}
void StanfordID::setID(int idNumber) {
	if (idNumber >= 0){
		this->idNumber = idNumber;
	}
}

// destructor
StanfordID::~StanfordID() {
	//释放内存
}
```
对于析构函数来说，它是为了释放我们在空间中使用的内存，但当我们`new`一个新对象时，它会自动帮我们释放的，尽管如此，这依旧很重要
### 6.4 Inheritance
- 多态性：不同的对象有可能会使用相同的接口
- 可扩展性：继承允许扩大类来创造子类
```cpp
// .h file
class Shape {
public:
	virtual double area() const = 0; // 虚拟函数
}；

class Circle : public Shape { // :继承
private:
	double _radius;
public:
	Circle(double radius): _radius{radius} {}; // 构造函数 uniform initialization
	double area() const {
		return 3.14 * _radius * _radius;
	} // 重写 area()函数
}
```
`public` `protected` `private`
## 7. Template Classes
### 7.1 预处理器与宏定义
在创建我们自己的vector的时候，会遇到对不同的类型写不同的代码的情况，但它们之间的区别又很小，只有一个类型的差别，例如
```cpp
class IntVector {
public:
	int& at(size_t index);
	void push_back(const int& elem);
private:
	int* elems;
	size_t logical_size;
	size_t array_size;
};

class DoubleVector
class StringVector
```
对于这种情况，我们引出了预处理器与宏
```cpp
#define GENERATE_VECTOR(MY_TYPE)
	class MY_TYPE##Vector { 
	public: 
		MY_TYPE& at(size_t index); 
		void push_back(const MY_TYPE& elem); 
	private: 
		MY_TYPE* elems; 
		size_t logical_size; 
		size_t array_size; 
};

GENERATE_VECTOR(int)
int Vector v1;
v1.push_back(5);
```
预处理器允许其在编译之前运行
通过这种方式，允许我们创建不同类型的vector
但对于这种方式，也会存在其缺点
- 类C语言的语法
- 很难做类型检查
- 有时会忘了做宏定义
因此，继续引出**Templates**
### 7.2 Templates
模板允许自动生成代码
```cpp
template <typename T> // 模板声明 T会自己替换类型
class Vector {
public:
	T& at(size_t index);
	void push_back(const T& elem);
private:
	T* elems;
}
```
对于特定类型的模板，我们称其为**Template Instantiation**，以下均为模板实例化
```cpp
Vector<int> intVec;
Vector<double> doubleVec;
Vector<std::string> strVec;
Vector<Vector<int>> vecVec;
struct MyCustomType {};
Vector<MyCustomType> structVec;
```
编译器会自己替换类型的
对于`typename`来说，允许替换
```cpp
template <typename T>
class Vector {};

template <size_t N>
class SizeTemplate {};

template <bool B>
class BoolTemplate {};

template <typename T, std::size_t N>
```
#### 7.2.1 一些模板的怪癖
1. 在`.cpp`文件中必须包含`template< >`
2.  `.h`必须在底部包含`.cpp`
```cpp
// .h
template <typename T>
class Vector {
public:
	T& at(size_t i);
}

#include "Vector.cpp" // 怪癖2 必须包含

// .cpp
template <typename T> // 怪癖1 显式写出来
T& Vetor<T>::at(size_t i) {

}
```
3. `typename`与`class`是一样的
```cpp
template <typename K, typename V> struct pair;
template <class K, class V> struct pair;
```
### 7.3 Const Correctness
当我们书写以下函数时，编译器会给出错误`No such method size!`
```cpp
void printVec(const Vector<int>& v) {
	for (size_t i = 0; i < v.size(); i++) {
		std::cout << v.at(i) << " ";
	}
	std::cout << std::endl;
}
```
但我们明明在之前就已经实现了`size`
```cpp
template<class T>
class Vector {
public:
	size_t size();
	bool empty();
	T& operator[] (size_t index);
	T& at(size_t index);
	void push_back(const T& elem);
};
```
问题在于
- 对于v来说传递的是个常数，不希望修改v
- 编译器不确定`size`是否会修改v
- 因为成员函数是有可能会修改v的
修改方法为，在后面加上`const`标识符，以告诉编译器，我确保此方法不会修改对象
```cpp
template<class T>
class Vector {
public:
	size_t size() const;
	bool empty() const;
	T& operator[] (size_t index);
	T& at(size_t index) const;
	void push_back(const T& elem);
};
```
对于`.cpp`文件也应该加上`const`标识符
```cpp
template <class T>
size_t Vector<T>::size() const {
	return logical_size;
}
```
#### 7.3.1 Const interface
- 对象被标记为const后只能使用const interface
- const interface是作为对象中的一个常量函数
#### 7.3.2 Const Cast
对于`T& at(size_t index) const;`来说存在两个错误
1. const可以修改
	- 返回一个引用常量`const T& at(size_t index) const;`
2. 非const不可以修改
	- 重写函数，写两个版本的`at()`，一个版本用于接收const参数，另一个用于接收非const参数
```cpp
template<class T>
class Vector {
public:
	const T& at(size_t index) const;
	T& at(size_t index);
}
```
但如果我们有许多函数要对常量与非常量进行运算，那每个函数就要写两遍，这过于冗余了，因此我们引出**casting**
- casting 将一个类型强制转换为另一个类型
- `const_cast` 将非const强转为const
`const_cast<Vector<T>&>(*this).findElement(value);`
- `const_cast`将非const强转为const
- `Vector<T>&`非const引用
- `*this`解引用`const Vector<T>*`
- `findElement`非const版本的`findElement()`
什么时候使用**const_cast**？
**最好不要**
## 8. Template Functions
### 8.1 Template Functions
#### 8.1.1 三元运算符
```cpp
return a < b ? a : b;
```
若 a < b 则返回 a 反之 b
当我们想将不同的类型进行上述比较，按之前的方法可能需要些若干个函数，分别表示不同的类型，但现在我们可以使用模板
```cpp
template <typename T>
T min(T a, T b) {
	return a < b ? a : b;
}
```
同样的，可以写出引用类型，防止复制一份额外的，占用内存
```cpp
template <typename T>
T min(const T& a, const T& b) {
	return a < b ? a : b;
}
```
#### 8.1.2 调用模板函数
- explicit instantiation（显式实例化）
```cpp
min<int>(106, 107);
min<double>(1.2, 3.4);
```
- implicit instantiation（隐式实例化）
```cpp
min(106, 107);
min(1.2, 3.4);
```
编译器会自己调用类型，有点像之前学到的`auto`关键字
若给出`min(106, 3.14)`则无法编译，解决方法为改变函数签名
```cpp
template <typename T, typename U>
auto min(const T& a, const T& b) {
	return a < b ? a : b;
}
```
### 8.2 Concepts
编译器发现错误只会在实例化之后，这也就是说，有些事物可以被实例化，但无法被比较，此时编译器会正常实例化，但无法比较，由此，如何才能在实例化之前发现错误（无法被比较）？
#### Introducing C++ concepts
创建Comparable concept
```cpp
template <typename T>
concept Comparable = requires(const T a, const T b) {
	{a < b} -> std::convertible_to<bool>;
};
```
- concept: a named set of constraints
- requires: 提供两个参数
- `{a < b}`(constraint): 任何在大括号内的东西都必须被无错误地编译
- `std::convertible_to<bool>`(constraint): 结果必须为布尔值
此时，就可以避免无法被比较的错误了
```cpp
template <typename T> requires Comparable<T>

// 简短写法
template <Comparable T>
```
### 8.3 Variadic Templates
如果我们想在一个函数里面接受更多的参数，应该怎么做，简单的想法是重写许多个函数，让编译器自己寻找该用哪一个，这显然过于冗余
**Templates + recursion**
```cpp
// base case
template <Comparable T>
T min(const T& v) {return v;}

// recrusive case
template <Comparable T, Comparable... Args>
T min(const T& v, const Args&... args) {
	auto m = min(args...);
	return v < m ? v : m;
}
```
- `Comparable... Args` **Variadic template**匹配0或更多个类型
- `const Args&` **Parameter pack**匹配0或更多个参数
- `min(args)` 替换为实际参数
### 8.4 Template Metaprogramming
**TMP is Turing complete**
如何让我们得到既可以在编译时运行又有可读性的代码呢？
**constexpr/consteval**(C++20)
constexpr: 尝试在编译时运行此段代码
consteval: 必须在编译时运行此段代码
```cpp
constexpr size_t factorial(size_t n) {
	if (n == 0) return 1;
	return n * factorial(n - 1);
}
consteval size_t factorial(size_t n) {
	if (n == 0) return 1;
	return n * factorial(n - 1);
}
```
## 9. Functions and Lambdas
### 9.1 Functions and Lambdas
#### 9.1.1 传递函数
有时会想把一个布尔函数作为查找一个东西的条件，我们在此称其为**predicates**那么如何优雅的将此条件加入到函数中呢，简单想法为作为函数中的一个参数来实现，问题在于其类型是什么，以`find_if()`为例
```cpp
template <typename It, typename Pred>
It find_if(It first, It last, Pred pred) {
	for (auto it = first; it != last; ++it) {
		if (pred(*it)) return it;
	}
	return last;
}
```
在此我们对于predicates的类型作为**typename**，编译器会自己帮我们找其类型具体是什么的。在函数内也传递了此条件，运用了此条件
旁注，Pred的类型具体是什么？
在C++中，这可以作为一个**函数指针**，用于指向一个函数
```cpp
Pred = bool(*)(int);
Pred = bool(*)(char);
```
但有时候又会遇见在条件中不止一个参数的情况，比如`bool isLessThan(int elem, int n)`在这种情况，我们就无法在其他函数中使用此函数，因为要在其他函数使用此函数，只能传递一个参数，由此引出**Lambda Functions**
#### 9.1.2 Lambda函数
Lambda函数允许我们在使用一个参数的情况下也依然可以比较多个参数之间的值，举例来说，
```cpp
auto lessThanN = [n](int x) {
	return x < n;
}
```
在这里我们实际上只传递了一个参数x
lambda函数允许出现在其他函数的参数中
#### 9.1.3 Functor(仿函数)
一个扮演着函数的对象，由`operator()`定义的对象
```cpp
template <typename T>
struct std::greater {
	bool operator()(const T& a, const T& b) const {
		return a > b;
	}
};
std::greater<int> g;
g(1, 2);
```
由于仿函数是一个对象，所以其存在state
```cpp
struct my_functor {
	bool operator()(int a) const {
	return a * value;
	}
	int value;
};
my_functor f;
f.value = 5;
f(10);
```
#### 9.1.4 仿函数与Lambda函数之间的关系
当我们写一个Lambda函数时，实际上就是在用仿函数来写，例如以下两函数表达的是同一个意思
```cpp
int n = 10;
auto lessThanN = [n](int x)
{ return x < n; };
find_if(begin, end, lessThanN);
```
```cpp
class __lambda_6_18
{
public:
	bool operator()(int x) const
	{ return x < n; }
	__lambda_6_18(int& _n) : n{_n}
{}
private:
	int n;
};
int n = 10;
auto lessThanN = __lambda_6_18{n};
find_if(begin, end, lessThanN);
```
### 9.2 Algorithms
`<algorithm>`是一个模板函数的集合
### 9.3 Ranges and Views
#### 9.3.1 Ranges
Ranges是一个STL的新版本
定义：range是任何拥有`begin`和`end`的东西
这意味着许多可以迭代的数据结构都存在ranges
#### 9.3.2 Views
一种组合算法的方法
定义：惰性的适配另一个range的range
若我们用现在的STL来写代码，可能会是这样的
```cpp
std::vector<char> v = {'a', 'b', 'c', 'd', 'e'};
// Filter -- Get only the vowels
std::vector<char> f;
std::copy_if(v.begin(), v.end(), std::back_inserter(f), isVowel);
// Transform -- Convert to uppercase
std::vector<char> t;
std::transform(f.begin(), f.end(), std::back_inserter(t), toupper);
// { 'A', 'E' }
```
但使用veiws来写，就会简单很多
```cpp
std::vector<char> letters = {'a', 'b', 'c', 'd', 'e’};
auto f = std::ranges::views::filter(letters, isVowel);
auto t = std::ranges::views::transform(f, toupper);
auto vowelUpper = std::ranges::to<std::vector<char>>(t);
```
同时还可以使用`|`链式操作
```cpp
std::vector<char> letters = {'a','b','c','d','e'};
std::vector<char> upperVowel = letters
| std::ranges::views::filter(isVowel)
| std::ranges::views::transform(toupper)
| std::ranges::to<std::vector<char>>();
```
使用ranges/views的优缺点
优点
- 不用担心迭代器
- 更好的错误消息提示
- 可读性更高、函数化更好的语法
缺点
- 太新了（C++20开始），很少人了解
- 缺乏编译器支持
- 与手工编码版本相比的性能缺失
## 10. Operator Overloading
### 10.1 Member overloading
当我们想要比较我们自己创建的类的时候，常常会因为此类无法被比较而失败，但C++允许我们自己**重载操作符**，以帮助我们比较两个类，就像我们之前学习的`StanfordID`类，就可以对其操作符进行重载，具体怎么做？
- 就像我们在类中声明函数一样，我们可以以声明函数的方式声明操作符
- 当我们对我们创建的对象使用操作符时，这代表在执行自定义的函数或操作符
- 就像函数重载一样，当我们给了一样的名字，它就会重写操作符的行为，编译器自己会找到该用哪个的
因此，我们的`StanfordID`的`小于操作符`可以写成
```cpp
bool StanfordID::operator< (const StanfordID& other) const {
	return idNumber < other.getIdNumber();
}
```
### 10.2 Non-member overloading
上文提到的是**member overloading**，不止这种方法，在C++中更被人们所喜欢和应用的是**Non-member overloading**，为什么？
1. 允许(left-hand-side)左边是非类类型(non-class type)
2. 允许重载我们没有掌控的类的运算符，比如我们可以将`StanfordID`与其他类进行比较
```cpp
// Non-member
bool operator< (const StanfordID& lhs, const StanfordID& rhs);
// member
bool StanfordID::operator< (const StanfordID& rhs) const {...}
```
### 10.4 `friend`
`friend`关键字允许non-member函数或non-member类访问其他类中private里的值
`friend`应该写在`.h`文件中显式写出来
```cpp
// .h file
class StanfordID {
private:
std::string name;
std::string sunet;
int idNumber;
public:
// constructor for our StudentID
StanfordID(std::string name, std::string sunet, int idNumber);
... 
friend bool operator <(const StanfordID& lhs, const StanfordID& rhs);
}

// .cpp file
#include StanfordID.h
bool operator< (const StanfordID& lhs, const StanfordID& rhs)
{
	return lhs.idNumber < rhs.idNumber;
}
```
**运算符允许传递函数不存在的类型**
- operator overloading解锁了我们定义对象新的功能和意义
- 运算符应该有意义，传递一些函数并不关乎类型本身
- 仅在需要的时候overload，比如不需要使用IO流的时候就不需要重写`<<`或`>>`
## 11. Move Semantics(移动语义)
假设现在我们存在一个`Photo`类
```cpp
class Photo {
public:
	Photo(int width, int height);
	Photo(const Photo& other);
	Photo& operator=(const Photo& other);
	~Photo();
private:
	int width;
	int height;
	int* data;
};

// Constructor
Photo::Photo(int width, int height)
	: width(width)
	, height(height)
	, data(new int[width * height])
{}

// Copy Constructor
Photo::Photo(const Photo& other)
	: width(other.width)
	, height(other.height)
	, data(new int[width * height])
{
	std::copy(other.data, other.data + width * height, data);
}

// Copy Assignment
Photo &Photo::operator=(const Photo& other) {
	// Check for self assignment
	if (this == &other) return *this;
	delete[] data; // Clean up old pixels!
	// Copy over new pixels!
	width = other.width;
	height = other.height;
	data = new int[width * height];
	std::copy(other.data, other.data + width * height, data);
	return *this;
}

// Destructor
Photo::~Photo()
{
	delete[] data;
}
```
当此时已经存在一个`Photo`实例时，现在想要再得到一个相同的`Photo`时，第一时间可能会想到使用Copy复制一份，但可以有更好的方式不需要通过复制来实现——移动语义（`std::move()`）。使用这种方式可以直接指向原`Photo`而不用复制。但是，使用这种方式一定是安全的吗？如果我们未来还需要再使用这个对象，那此时原对象已经指向null了，就会出现空指针异常的情况。
### 11.1 左值与右值
关于左值与右值，上文已经有提过，在此需要对其加深理解。
- 左值的生命周期在**范围**结束
- 右值的生命周期在**本行**接受
- 左值是**持续值**
- 右值是**暂时值**
对于引用类型来说，
- 左值：`Type&` 
- 右值：`Type&&`
**重载`&` `&&`以区分左值引用与右值引用**
### 11.2 移动语义
- Move constructor
	- `Type::(Type&& other)`
- Move assignment operator
	- `Type& Type::operator=(Type&& other)`
一般来说，编译器会决定使用`&`还是`&&`，但这一定是最有效率的吗？事实上，若我们知道一个值就是**临时的**，不妨主动使用移动语义。
要想显式删除移动语义，可以使用
- Move constructor
	- `Type::(Type&& other) = delete;`
- Move assignment operator
	- `Type& Type::operator=(Type&& other) = delete;`
### 11.3 SMFs
目前为止，我们已经学习了许多Special Member Functions
- `Type::Type(const Type& other);`
- `Type& Type::operator=(const Type& other);`
- `Type::Type(Type&& other);`
- `Type& Type::operator=(Type&& other);`
- `~Type::Type();`
但我们在写每个类的时候都需要把他们都写出来吗？
不需要
- Rule of Zero
若类不需要管理内存（或其他外部资源），编译器会创造有效的SMFs的
- Rule of Three
若类需要管理内存，必须定义`copy assignment/constructor`即`Destructor` `Copy Assignment` `Copy Constructor`，若不定义这些，编译器不会复制底层资源
- Rule of Five
若定义了`copy assignment/constructor`即`Destructor` `Copy Assignment` `Copy Constructor`，则也应该定义`move constructor/assignment`，即`Move Assignment` `Move Constructor`
这不是必须的，但若不定义，则代码可能会在包含一些不必要的复制时变得缓慢
## 12. std::optional & type safety
### 12.1 Type safety
```cpp
int div_3(int x) {
	return x / 3;
}

div_3("hello")
```
这就是一个类型错误，函数需要的是`int`类型的参数，但给的是个`string`，编译器会给出错误，永远不会运行。
### 12.2 std::optional< T >
`std::optional`是一个模板类要么包含一个类型为T的值，要么什么也不包含，对于不包含的情况会表示为**nullopt**
nullopt是一个可以转换为任意可选类型的值的对象
```cpp
void main(){
	std::optional<int> num1 = {}; //num1 does not have a value
	num1 = 1; //now it does!
	num1 = std::nullopt; //now it doesn't anymore
}
```
`std::optional`含有以下接口
- `.value()`:
returns the contained value or throws `bad_optional_access` error
- `.value_or(valueType val)`
returns the contained value or default value, parameter `val`
- `.has_value()`
returns true if contained value exists, false otherwise
优点
- 函数签名创建更多有意义的contracts
- 调用类函数时，会有更多保证和有效的行为
缺点
- 需要在任何地方使用`.value()`
- 有可能存在`bad_optional_access`
- optionals会有未定义的行为（错误检查）
- 有很多情况我们想要`std::optional<T&>`这个引用类型，但是C++并没有提供
### 12.3 总结
- 可以保证程序在一个严格的类型检查下进行
- `std::optional`是一个工具可以用`.has_value()`来返回一个值或nothing
- 这个应用并不广泛并且慢，所以C++大多数数据结构并没有`optional`
- 不仅在类中使用，在应用程序代码中也可以使用，并且是很推荐的。
## 13. RAII, Smart Pointers, Building Projects
### 13.1 RAII(Resource Acquisition Is Initialization)资源获取即初始化
#### 13.1.1 Exceptions
- 异常是一种处理错误的方法，用于抛出异常
- **thrown**
- 在非严重错误时捕捉异常用于继续代码进行
#### 13.1.2 RAII
什么是RAII？
- 一个类中所有使用的资源应该被构造函数所获取
- 一个类中所有使用的资源应该被析构函数所释放
为什么使用RAII？
- 通过遵守RAII政策，可以避免**half-valid**状态
- 无论怎样，析构函数总是会被调用在资源用完之后
- 资源/对象会立即运用当它们被创建
### 13.2 Smart Pointers
对于RAII for locks，我们会用`lock_guard`，这会创建一个对象用于在构造函数中获取所有资源，并在析构函数中释放这些资源
而对于RAII for memory，我们也做同样的事情，只不过在这里我们称之为**smart pointers**
- `std::unique_ptr`
只属于这一个资源，不可以被复制
- `std::shared_ptr`
可以复制，当底层内存超出范围时，就会被破坏
- `std::weak_ptr`
一类旨在缓解循环依赖的指针
```cpp
// 不要这么声明
Node* n = new Node;

std::unique_ptr<T> uniquePtr = std::make_unique<T>();

std::shared_ptr<T> sharedPtr = std::make_shared<T>();

std::weak<T> wp = sharedPtr;
```
一定要使用`std`来make smart pointer
- 如果不这么做，会分配两次内存，一次是为了声明一个指针，另一次是为了new一个对象
- 整个代码应该保持一个连续性，更易读
### 13.3 Building C++ Projects
#### 13.3.1 编译流程
编写一个C++代码后，它要被翻译为机器代码以使编译器识别
`g++ main.cpp -o main`
#### 13.3.2 Make
**make**是一个构建系统程序来帮助我们编译
- 你可以规定编译器做什么
- 为了使用**make**需要有一个**Makefile**
```makefile
# Compiler
CXX = g++
# Compiler flags
CXXFLAGS = -std=c++20
# Source files and target
SRCS = $(wildcard *.cpp)
TARGET = main
# Default target
all:
	$(CXX) $(CXXFLAGS) $(SRCS) -o $(TARGET)
# Clean up
clean:
	rm -f $(TARGET)
```
#### 13.3.3 CMake
**CMake**是一个构建系统生成器，可以用CMake来生成Makefiles，就像一个Makefiles的高一层抽象
对于一个`CMakeList.txt`文件来说，存在以下内容
```txt
cmake_minimum_required(VERSION 3.10)
project(cs106l_classes)
set(CMAKE_CXX_STANDARD 20)
file(GLOB SRC_FILES "*.cpp")
add_executable(main ${SRC_FILES})
```
- `set`为设定编译器版本为C++20
- `GLOB`告诉CMake搜索所有后缀为`cpp`的文件
- `add_executable`该命令将程序的所有源文件添加到可执行文件中
如何使用CMake？
1. 拥有一个`CMakeLists.txt`
2. 创建一个`build`文件夹
3. 运行CMake
4. 运行make
5. 执行文件`./main`