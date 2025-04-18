---
title: CS106L Winter2025 Assignment4 Ispell
description: 关于CS106L的第四个作业的个人解法
date: 2025-04-18
categories:
    - CS106L
    - 解题思路
tags:
    - C++
    - 学习笔记
comment: true
---

## 写在前面
本文取自Stanford CS106L Winter2025 Assignment4: Ispell 本任务主要要求我们实现一个拼写检查程序，即若我们给出的单词有误会给出较为合理的建议。主要用到的知识有容器、Lambda函数等内容。
## 具体思路
### 题目大意
1. 将所有常见的英语单词字典添加到内存中，若在字典中找不到该单词则认为拼写错误
2. 若可以通过添加、删除、替换、交换等操作且此操作只需要进行一次的，则将其添加到建议列表中
3. 不能使用for/while循环
### `tokenize`
#### 题目大意
此方法最终会实现一个将String分割成一系列Token，对于一个Token来说其作为一个结构体，包括`struct Token { std::string content; size_t src_offset; };`
假设现有一字符串`history will absolve me`，我们可以将其以空格为边界分成四个Token
- `{ content: "history", src_offset: 0 }`
- `{ content: "will", src_offset: 8 }`
- `{ content: "absolve", src_offset: 13 }`
- `{ content: "me", src_offset: 21 }`
可以看出，`content`会以第一个字母为起点，空格为终点存储一个单词，`sec_offset`为此第一个字母在整个字符串中的索引，其中需要包括空格。
#### 代码实现
##### **Identify all iterators to space characters**
此任务需要我们找出所有空白字符并将其作为迭代器保存到vector中，题目给了我们函数`find_all(Iterator begin, Iterator end, UnaryPred pred)`，它可以返回此迭代器中所有符合`pred`条件的所有迭代器并将其保存在vector中，那么我们就需要用参数`source`来作为迭代器，String类提供了作为迭代器的接口，`begin` `end`，条件题目也给了我们判断此字符是否为空格的函数`isspace`，可以写出代码，
```cpp
  auto space = find_all(source.begin(), source.end(), isspace);
```
##### **Generate tokens between consecutive space characters**
目前，我们已经拥有了所有空字符的迭代器，现在需要我们将两个空字符迭代器之间的内容保存为一个Token，Token的迭代器会自动处理两边的空白，只需要提供起始迭代器与终止迭代器即可，此时就可以找到一个Token，但其中不止存在一个Token，所以我们还需要遍历整个字符串，但题目要求不能使用for/while循环，所以使用另一个函数
`std::transform(InputIt1 first1, InputIt1 last1, InputIt2 first2, OutputIt d_first, BinaryOp binary_op)`
此函数参数可以分为三个部分
- `first1` `last1` `first2`，对于此输入范围，函数要求给定两个相等的输入范围，一个从`first1`开始到`last1`结束，另一个从`first2`开始
- `d_first` 将结果存储到`d_first`中，大小与上文输入范围相同
- `binary_op` 应用于两个部分
对于`binary_op`应创建一个lambda函数，其中参数为两个迭代器，和字符串，以创建Token，其中source应按引用传递，
对于`d_first`应先创建一个集合以存储我们创建的Token，再使用`std::inserter`函数以保存创建的Token
对于输入范围，应选择两个连续的迭代器，以遍历所有迭代器
```cpp
  std::set<Token> tokens;
  std::transform(space.begin(), space.end() - 1, space.begin() + 1, 
                  std::inserter(tokens, tokens.end()),
                  [&source] (auto it1, auto it2) {
                    return Token { source, it1, it2};
                  }
  );
```
##### **Get rid of empty tokens**
此时我们所创建的Token可能有一些Token是含有空白字符的，我们要将其删掉，
`std::erase_if(std::set<Key, Compare, Alloc>& c, Pred pred)`
对于c，传递我们上文给的集合tokens即可
对于pred，创建一个lambda函数，以判断token是否为空
```cpp
  std::erase_if(tokens, 
                [] (Token token) {
                  return token.content.empty();
                }
                );
```
### `spellcheck`
#### 题目大意
该方法接受`tokenize`所得到的token来进一步给出建议，并返回一个`Mispelling`，其中包括一个错误的Token，和一系列建议的单词
大致过程为
1. 跳过正确拼写的单词
2. 查找距离正确单词仅“一步之遥”的单词
3. 删除没有建议的错误拼写单词
#### 代码实现
##### **Skip words that are already correctly spelled.**
若一个单词已经存在在了dictionary中了，我们就认为它是正确的，思路很简单，在此处需要学习的是`std::ranges::views::filter`，在此笔者使用题目给的较为简洁的方法，即使用namespace来简洁化
```cpp
namespace rv = std::ranges::views;
auto view = source | rv::filter(/* A lambda function predicate */);
```
我们的任务只需要填写这个lambda函数，此函数参数为Token，lambda参数为dictionary，此处的dictionary应按引用传递，来判断此Token是否在dictionary中
```cpp
auto view = source | rv::filter([&dictionary] (Token token) {
    return !dictionary.contains(token.content);
  });
```
##### **Find one-edit-away words in the dictionary using Damerau-Levenshtein**
根据题目提示，此时我们应该与上题相结合
```cpp
namespace rv = std::ranges::views;
auto view = source 
    | rv::filter(/* A lambda function predicate */)
    | rv::transform(/* A lambda function taking a Token -> Mispelling */);
```
此lambda函数应接受一个Token，返回距离为一的所有单词，求距离是否为一，使用题目给的函数`levenshtein`来判断即可
最后需要返回一个Mispelling对象
```cpp
	| rv::transform([dictionary] (Token token) -> Mispelling {
    auto view = dictionary | rv::filter([token] (std::string str) {
      return levenshtein(token.content, str) == 1;
    });
    std::set<std::string> suggestions(view.begin(), view.end());
    return Mispelling { token, suggestions };
  });
```
##### **Drop misspellings with no suggestions.**
在此我们需要找到没有办法进行任何修改使其变为正常单词的单词了，故此处还需要使用`filter`来判断是否符合此情况，过滤完成后返回最新的Mispelling
```cpp
 | rv::filter([] (Mispelling miss) {return !miss.suggestions.empty();});
  return std::set<Mispelling> (view.begin(), view.end());
  };
```
[GitHub地址](https://github.com/YuTaki23/CS106L-Winter-2025/tree/main/assign4)