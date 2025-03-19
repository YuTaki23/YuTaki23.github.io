---
title: CS61A SP24 Scheme
description: 关于CS61A的最后一个项目Scheme的个人解法
date: 2025-03-07
image: image1.png
categories:
    - CS61A
    - 解题思路
tag:
    - Python
    - 学习笔记
comment: true
---

## 写在前面

本项目为UC Berekly CS61A SP24的 Project4: Scheme。以下为个人对此项目的见解，如有错误或更好的解决方案，欢迎与我联系。

[点这里向我发邮件~](mailto:yutaki23@163.com)

## 具体思路

### Problem 1

首先通读一下题目，此题需要我们完成两个函数`define`和`lookup`，同时给了我们关于`Frame`的一些信息，`Frame`有两个instance attribute。

1. `bindings` 这是一个字典，回顾一下字典的相关内容，字典，相当于其他语言的map，存在一个key和一个value，每个字典当中只能有一个唯一的key，但key中存放的数据可以是相同的。此题中，key就代表的是symbol，而value代表的就是其本意，相当于我们在python当中的等号赋值用法，例如，`x = 3`在这里key就是x，value就是3。
1. `parent` 每个frame都可以被嵌套在另一个frame里面，而他们最终的frame都是None。

- `define`

非常显而易见了，上文中也提到过，在此不再赘述。

```python
def define(self, symbol, value):
    """Define Scheme SYMBOL to have VALUE."""
    self.bindings[symbol] = value
```

- `lookup`

此函数可以想成三个case，题目当中也给了我们很详细的描述。

1. 若symbol在当前frame里，直接返回value。
2. 若symbol不在当前frame里，且其含有父frame，且父frame里含有此symbol，返回value。关于这个case，有几点需要说明
    - 其含有父frame。问：如何界定此frame是否合法？答：只要不为None，都是合法的。
    - 父frame里含有symbol。问：如何访问父frame？答：用点表达式访问parent即可。
3. 若均不满足以上情况，即symbol不在当前frame里，且没有父frame，arise error。

```python
    def lookup(self, symbol):
        """Return the value bound to SYMBOL. Errors if SYMBOL is not found."""
        current_frame = self
        # case1 若symbol在当前frame
        if symbol in current_frame.bindings:
            return current_frame.bindings.get(symbol)
        # case2 symbol不在当前frame 但在parent frame
        else:
            current_frame = current_frame.parent
            while current_frame is not None:
                if symbol in current_frame.bindings:
                    return current_frame.bindings.get(symbol)
                current_frame = current_frame.parent
        # case3 都不存在 引发error
        raise SchemeError('unknown identifier: {0}'.format(symbol))
```

### Problem 2

首先通读一下题目，此题要求我们实现一个`scheme_apply`函数中的一个case，即`BuiltinProcedure`。在完成一个表达式时，存在内置的一些操作如 `+ - * /`等，在此，就需要我们实现这些操作。

`BuiltinProcedure`含有两个instance attribute。

1. `py_func` 找到对应的操作符号，用python的内置函数来实现表达式，不用我们自己实现（自己似乎也实现不了，有点不切实际）。
2. `need_env` 有些内置函数需要特定的环境。

此题可分为三个步骤来写，题目中也给的很清楚。

1. 将`scheme list` 转变为`python list`，与上题很类似。
2. 判断`need_env`的是否，很简单。
3. 使用***args表示法**来调用`py_func`这里可能需要了解一下什么是 *args表示法。

```python
        try:
            # 将scheme list转变为 python list
            lst = []
            while args is not nil:
                lst.append(args.first)
                args = args.rest
            # 判断 need_env
            if procedure.need_env is True:
                lst.append(env)
            # 调用 py_func
            return procedure.py_func(*lst)
        except TypeError as err:
            raise SchemeError('incorrect number of arguments: {0}'.format(procedure))
```

### Problem 3

此题需要让我们将`scheme_eval`补充完整，大致题目已给出，我们需要完成的是判断此表达式，并通过上文完成的`scheme_apply`函数来计算结果。题目还是给了我们大致思路。

1. 判断**operator**，使用递归。*Recrusion*，即自身调用自身，那一定是在`scheme_eval`函数里调用`scheme_eval`。对于此函数，我们需要三个参数，`expresson` `environment`以及一个默认参数，文中给了我们代码提示`first = expr.first`，由此即可以判断此处的`expression = first`，同时还要求我们**evaluate to a procedure instance**。`scheme_apply`的开头就给了我们提示
2. 判断**operands**，依旧使用递归。与上文类似，题目已经给了我们`rest = expr.next`，同时题目要求我们收集到scheme列表里，下文有提示，使用`Pair`的`map`方法，观察`map`方法，这提供了一个形参`fn`，显然此处的`fn`就是`scheme_eval`，但`scheme_eval`存在两个形参，这就需要我们进行转换，使用**lambda函数**。
3. 之后就是调用`scheme_apply`，很显然，在此不做过多赘述。

```python
else:
    # evaluate operator
    procedure = scheme_eval(first, env)
    validate_procedure(procedure)
    # evaluate operand
    operands = rest.map(lambda x: scheme_eval(x, env))
    return scheme_apply(procedure, operands, env)
```

### Problem 4

此题要求我们实现`define`，简单来说，就是将一个**expressions**赋值给一个**symbol**。我们在这里只需要实现第一个部分，即计算一个表达式作为**expressions**并复制给一个提供给我们的**symbol**，下方还有提示，需要我们用到`Frame`里的`define`。那我们想一想`define`函数需要我们提供什么参数呢？

1. `symbol` symbol总是存在于一个表达式的第一位，函数也给了我们一个提示，即`signature = expressions.first`，由此我们就可以看出我们的symbol就是这个。
2. `expressions` 对于一个表达式来说，我们需要赋值的不仅仅是表达式本身，还是它计算后的数值是什么，所以在这里我们还需要用到上文完成的`scheme_eval`函数。
3. 最后，需要我们返回已绑定的`symbol`。

```python
signature = expressions.first
if scheme_symbolp(signature):
    # assigning a name to a value e.g. (define x (+ 1 2))
    validate_form(expressions, 2, 2) # Checks that expressions is a list of length exactly 2
    env.define(signature, scheme_eval(expressions.rest.first, env))
    return signature
```

### Problem 5

此题向我们描述了`quote`的用法，在此不做过多赘述，详情可看题目要求。在此仅讲讲需要实现的功能。

给出一段`expressions`，需要我们原封不动的返回，但对于`Pair`来说，不返回**nil**，由此可见，需要返回的是`expressions.first`。

```python
validate_form(expressions, 1, 1)
return expressions.first
```

**Congratulation! 第一部分已完成**

### Problem 6

此题我们需要完成Scheme中的一个特殊形式`begin`。

> `begin` 用于按顺序组合多个表达式，并返回最后一个表达式的值。例如，
>
> ```scheme
> (begin
>   (display "Hello")  ; 执行第一个表达式（输出 "Hello"）
>   (display "World")  ; 执行第二个表达式（输出 "World"）
>   (+ 1 2))          ; 返回最后一个表达式的值 3
> ```

那由此我们可以观察到，它会将所有表达式都执行一遍，但只返回最后一个表达式的值，是不是有点像我们说的递归的概念，不断的完成一个动作，直到最后一个动作再返回。由此我们就可以写出代码。

1. 首先有一个 `base case` 若此表达式为空，返回`None`。
2. 开始递归。首先我们想一想，想要evaluate一个表达式，应该怎么做？答：用上文完成的`scheme_eval`函数。
3. 如何判断是否到达了递归终点？对于一个`expressions`来说，它就像一个链表，最后一项的`rest`总是`nil`，这就到达了递归终点，也就是我们说的`base case`。
4. 对于`recrusion case` 来说，我们要不断的调用自身，直到到达`base case`。

```python
if expressions is nil:
    return None

exp = scheme_eval(expressions.first, env)
if expressions.rest is nil:
    return exp
else:
    return eval_all(expressions.rest, env)
```

### Problem 7

此题依旧需要我们完成Scheme中的一个特殊形式`lambda`。

>`lambda` 用于定义匿名函数，此题要求我们逐步计算表达式，并返回最后一个表达式的值。

其中，`LambdaProcedure`这个类存在一个实例`body`，存放的是scheme list，而实例`formals`存放的是正确的Pair嵌套表达式。

我们需要做的是创建并返回一个`LambdaProcedure`实例，那整体思路就很明朗的，再加上此函数还有给我们的提示，使用特定的语法知识，易得。

```python
    body = expressions.rest
    return LambdaProcedure(formals, body, env)
```

### Problem 8

此题需要我们创建一个新的`Frame`，其中含有两个参数，`formals` `vals`，均为scheme list，需要我们将每一个formal对应到val，且formals于vals的数量都是相等的，一一对应，直到达到scheme list的终点即可。值得注意的是，不是简单的复制父Frame里的内容，笔者在这里第一次就想错了，总的来说，题目已经告诉我们具体做法，在此不做过多赘述。

```python
        child_frame = Frame(self)
        while formals is not nil:
            child_frame.define(formals.first, vals.first)
            formals = formals.rest
            vals = vals.rest
        return child_frame
```

### Problem 9

此题需要我们完成`scheme_apply`里的一个`LambdaProcedure case`，题目已经给出提示，即

1. 创建一个新Frame，注意到，要想创建一个新的Frame，就应该想到用谁来创建，不能是Procedure本身，因为`make_child_frame`是在Frame类中实现的，所以应该找到一个Frame，即`Procedrure.env`
2. 将形参绑定到参数值，想一想，对于一个`Procedure`来说，存在两个实例，即`formals`和`body`，而`formals`对应的就是`make_child_frame`中的`formals`，而`body`对应的就是`eval_all`中的`expressions`。
3. 在当前Frame进行计算，也就是说此时的`env`应该变成我们创建的新Frame

综上易得，

```python
        child_frame = procedure.env.make_child_frame(procedure.formals, args)
        return eval_all(procedure.body, child_frame)
```

### Problem 10

此题需要我们完成`define`的一种新类型，题目已经给了详尽的步骤，在这里讲几个笔者踩到的坑。

1. 想清楚`symbol` `formals` `body`分别代表什么，首先想明白，signature与expression之间的关系，signature代表的是expression的第一个内容，也就是函数名，而其中`symbol`也叫函数，也就是`signature.first`，之后再想想`formals`，就是除了define和函数名剩下的内容，而`body`则代表的是除了define之外的所有内容，想清楚这五个之间的关系，对于这道题十分关键。
2. 题目告诉我们可以使用`do_lambda_form`，但笔者试了很多次也没有找到方法，若有人有更好的意见欢迎与我联系。
3. 在找到正确的`formals`后还需要使用`validate_formals()`函数来判断其正确性。

综上可知，

```python
        symbol = signature.first
        formals = signature.rest
        validate_formals(formals)
        body = expressions.rest
        env.define(symbol, LambdaProcedure(formals, body, env))
        return symbol
```

### Problem 11

这道题看着题目很长很难，但其实是纸老虎，根据它的题意来即可。

1. 看看`MuProcedure`可以发现需要两个参数，`formals`和`body`，`formals`题目已经给出，指的是expressions的第一个内容，而`body`就是除了第一个内容的其他内容。
2. 完成`scheme_apply`的`mu case`，这个内容跟上文的`lambda case`如出一辙，有区别的是，Frame的区别，因为`mu`是动态的，所以它的Frame一直在变，就不需要再用Procedure的Frame了。

综上可知，

```python
return MuProcedure(formals, expressions.rest)
```

```python
        child_frame = env.make_child_frame(procedure.formals, args)
        return eval_all(procedure.body, child_frame)
```

**Part 2 已完成！**

### Problem 12

这道题需要我们完成两个函数`do_and_form`和`do_or_form`，这两个函数从形式上来说都很相似，所以放在一起做，读题目可知，expressions会给出不定量个内容，我们需要判断每个内容的正确性来判断应返回什么，值得注意的是，对于`and`和`or`来说，存在一个关键的地方**短路**，每当遇到一个合适的内容时，就会舍弃后面的所有内容，仅返回当前内容。

因此，对于此题来说，递归是个很好的办法，具体步骤如下。

1. `base case 1` 若expression为空，则返回特定值。
2. `base case 2` 每当我们判断出来的当前内容的正确性，则直接返回。
3. `base case 3` 当所有内容都判断完毕后还没有找到正确性，则返回最后一个内容。
4. `recrusion case` 不断对下一个内容执行当前函数。

代码如下，

```python
def do_and_form(expressions, env):
    if expressions is nil:
        return True
    
    currency = scheme_eval(expressions.first, env)
    if is_scheme_true(currency):
        if expressions.rest is nil:
            return currency
        else:
            return do_and_form(expressions.rest, env)
    else:
        return currency
```

```python
def do_or_form(expressions, env):
    if expressions is nil:
        return False
    
    currency = scheme_eval(expressions.first, env)
    if is_scheme_false(currency):
        if expressions.rest is nil:
            return currency
        else:
            return do_or_form(expressions.rest, env)
    else:
        return currency
```

### Problem 13

此题需要我们完成`cond`，题目也给了我们描述。返回第一个表达式为`true`的值，若都不是正确的，那么就返回`else`表达式也就是最后一个表达式的值，这也是一个递归步骤，因此可知步骤为，

1. `base case` 判断到最后一个表达式，若还没有判断出来，则返回此表达式的值。
2. `recrusion case` 从第一个表达式开始逐步判断。

```python
            if clause.rest is nil:
                return test
            else:
                return eval_all(clause.rest, env)
```

### Problem 14

**这道题确实没看懂什么意思，在此也不误人子弟了。**

[点这里看总的代码实现](https://github.com/YuTaki23/CS61A-SP24)

## 写在后面

后续的题目就不再做了，是关于Scheme的。

作为UCB的CS61系列的第一门课，在当时还是初学者的我留下了很深的心理阴影，Scheme这个项目也一直停着没有做，这次重新捡回来，也算是给自己的一个交代。

至此，CS61A结束。

<p align = "right">YuTaki</p>

<p align = "right">2025年3月7日写于博学楼</p>