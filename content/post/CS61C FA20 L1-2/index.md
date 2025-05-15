---
title: CS61C FA20 Lab01-Lab02
description: 关于CS61C第1-2个实验的个人解法
date: 2025-05-15
categories:
    - CS61C
    - 解题思路
tags:
    - C
    - 学习笔记
comment: true
---

# Lab 01
## Exercise 1: See what you can C

本题只需要让我们修改 4 个全局变量来实现一些特定功能，题目给了我们 `for循环` `switch语句` `三元运算符` `if语句` 来让我们分别实现最终效果。

首先需要打印三个 `Happy`，这是关于 for 循环的，改变 `V0` 的值，让循环可以循环三次即可。

之后需要打印 `Yoshua`，这是关于 switch 语句的，这里出现了两个 `Yoshua`，但 `case 0` 这个情况没有 `break` 无法返回，会继续进行下去，所以不能用。

最后就是打印 `Go BEARS`，三元运算符是要括号内为真会返回第一个内容，为否返回第二个内容；if 语句是要括号内为真返回 if 下的语句，而 C 语言中为真只有 `1`

```c
#define V0 3
#define V1 3
#define V2 1
#define V3 3
```

## Exercise 2: Catch those bugs!

本题让我们使用 **gdb** 的一些操作来 debug 程序，看资源给的内容可以简单了解 gdb 的一些语法知识，一步一步往下调试即可。

```c
1. run arglist
2. b function
3. n
4. s
5. c
6. p
7. display
8. info locals
9. quit
```

## Exercise 3: Debugging w/ YOU (ser input)

本题给我们提供的一个情况是当程序中需要我们输入内容才能继续进行时，应如何调试。

给我们的提示是使用**重定向**

```bash
./a.out < fileName.txt
```

不光是在正常运行时可以使用重定向，在调试的时候也可以使用重定向。

比如我创建了一个 `name.txt` 的文件，存放我的名字 `yutaki`，此时运用重定向的方法就可以继续进行下去了。

## Exercise 4: Valgrind’ing away

本题向我们介绍了**valgrind**这个工具，这是用来检测程序中是否由内存方面的问题的一个工具。

对于第一个程序，数组中已经限制了 20 个，但 for 循环中没有限制结束标志，所以它会一直进行下去，导致内存泄漏，这回应发 `segmentation fault `。

## Exercise 5: Pointers and Structures in C

本题让我们判断一个链表是否存在循环，题目给的步骤已经十分详尽了，其中值得注意的是，

**Advance hare by two nodes. If this is not possible because of a null pointer, we have found the end of the list, and therefore the list is acyclic.**

无法推进节点，意味着即无法推进第一个节点也无法推进第二个节点。

```c
int ll_has_cycle(node *head) {
    node *tortoise = head;
    node *hare = head;
    
    while (1) {
        if (hare -> next == NULL || hare -> next -> next == NULL) {
            return 0;
        }
        hare = hare -> next -> next;
        tortoise = tortoise -> next;
        if (hare == tortoise) {
            return 1;
        }
    }
    return 0;
}
```

# Lab 02

## Exercise 0: Makefiles

本题向我们介绍了关于 Makefiles 的一些知识，当我们的项目十分庞大时，一个一个用 `gcc` 来编译就会很慢很麻烦，因此出现了 Makefiles，他能用一系列操作减少这些步骤的进行。

```c
1. clean
2. all
3. CC=
4. CFLAGS=
5. $(F00)
6. macOS
7. 18
```

## Exercise 1: Bit Operations

### `get_bit()`

获取一个二进制数中的某一个位，可以采用设置一个**掩码**，将这个位设置为 1，其他均为 0。再将原数字与掩码按位与，因为掩码其他都是 0，只有这一个是 1，所以只会保留这一个值。

```c
unsigned get_bit(unsigned x,
                 unsigned n) {
    unsigned mask = 1 << n;
    unsigned res = x & mask;
    return res >> n;
}
```

### `set_bit()`

要将一个二进制中的某一个位设置为一个特定值，一般来说，会想到分情况讨论，
首先还是设置一个掩码，在特定位上设置为 1，其他均为 0。
- 要设置成 1，使用按位或，这样不论原数字是 1 还是 0 都会保留 1
- 要设置成 0，使用按位与，将掩码取反，这样只有特定位是 0，其他都是 1，按位与只会得到 0
但在这里无法使用 `if` 操作，所以要想别的办法。

首先还是设置一个掩码，再将原数字的特定位变为 0，其他位不变，再将我们要设置的值左移特定位数，此时这个特定位存在两个掩码，一个是一定为 0，一个是要么 1 要么 0，那么此时再进行按位或操作，就可以得到答案

```c
void set_bit(unsigned * x,
             unsigned n,
             unsigned v) {
    unsigned mask = 1 << n;
    *x = *x & (~mask);
    unsigned target = v << n;
    *x = *x | target;
}
```

### `flip_bit()`

要反转特定为将 0 变为 1，或将 1 变为 0

设置一个掩码，使用按位异或

```c
void flip_bit(unsigned * x,
              unsigned n) {
    unsigned mask = 1 << n;
    *x = *x ^ mask;
}
```

## Exercise 2: Linear Feedback Shift Register

本题需要我们完成 LFSR 的**下一次迭代**，根据规则，LFSR 首先在最高位，一步一步慢慢到最低位。

关于这个步骤，根据图示可知，是用原寄存器与位 0，位 2，位 3，位 5 分别求按位异或。

那么如何求这个位呢？要用到上文中掩码的概念，用掩码与对应位取按位与操作来提取特定位。

最后移位，将寄存器右移一位并将反馈位放到最高位

```c
void lfsr_calculate(uint16_t *reg) {
    uint16_t lfsr = *reg;
    uint16_t bit0 = (lfsr >> 0) & 1;
    uint16_t bit2 = (lfsr >> 2) & 1;
    uint16_t bit3 = (lfsr >> 3) & 1;
    uint16_t bit5 = (lfsr >> 5) & 1;
    uint16_t feedback = (bit0 ^ bit2 ^ bit3 ^ bit5) & 1;
    lfsr = (lfsr >> 1) | (feedback << 15);
    *reg = lfsr;
}
```

## Exercise 3: Memory Management

本题要求我们实现一个**可变数组**，大致框架都给出了，我在这里讲几个我遇到的点

1. 在分配 `retval` 的内存时，首先要分配 `retval` 本身的内存，还要分配它里面属性的内存，但是 `size` 是不用分配的，在之后他也不会使用动态内存，简单设置为初始值即可。
2. 这是一个可变数组，其中 `size` 代表的是此数组中有几个值，`data` 代表的是整个数组，也就是说 `data` 实际上就是一个数组，它可以用 `[]` 来访问内容。
3. 当我们新设置的内容超过了原内存，要重新分配内存，用 `realloc`。
```c
vector_t *vector_new() {
    /* Declare what this function will return */
    vector_t *retval;

    /* First, we need to allocate memory on the heap for the struct */
    retval = malloc(sizeof(vector_t));

    /* Check our return value to make sure we got memory */
    if (retval == NULL) {
        allocation_failed();
    }

    /* Now we need to initialize our data.
       Since retval->data should be able to dynamically grow,
       what do you need to do? */
    retval->size = 1;
    retval->data = malloc(sizeof(int*));

    /* Check the data attribute of our vector to make sure we got memory */
    if (retval->data == NULL) {
        free(retval);				//Why is this line necessary?
        allocation_failed();
    }

    /* Complete the initialization by setting the single component to zero */
    retval->data[0] = 0;

    /* and return... */
    return retval;
}

int vector_get(vector_t *v, size_t loc) {

    /* If we are passed a NULL pointer for our vector, complain about it and exit. */
    if(v == NULL) {
        fprintf(stderr, "vector_get: passed a NULL vector.\n");
        abort();
    }

    /* If the requested location is higher than we have allocated, return 0.
     * Otherwise, return what is in the passed location.
     */
    if (loc < v->size) {
        return v->data[loc];
    } else {
        return 0;
    }
}

void vector_delete(vector_t *v) {
    free(v->data);
    free(v);
}

void vector_set(vector_t *v, size_t loc, int value) {
    /* What do you need to do if the location is greater than the size we have
     * allocated?  Remember that unset locations should contain a value of 0.
     */

    if (v == NULL) {
        fprintf(stderr, "vector_set: passed a NULL vector.\n");
        abort();
    }

    if (v->size <= loc) {
        int *new_data = realloc(v->data, sizeof(int*) * (loc + 1));
        if (new_data == NULL) {
            allocation_failed();
        }

        for (size_t i = v->size; i < loc; i++) {
            new_data[i] = 0;
        }
        v->data = new_data;
        v->size = loc + 1;
    }
    v->data[loc] = value;
}
```