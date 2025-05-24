---
title: CS61C FA20 Lab03-Lab04
description: 关于CS61C第3-4个实验的个人解法
date: 2025-05-24
categories:
    - CS61C
    - 解题思路
tags:
    - C
    - 学习笔记
comment: true
---

# Lab 03
## Exercise 1

本题让我们熟悉了 venus 的用法，这是一个可以模拟 riscv 代码的一个编辑器。

1. `.data` 表示数据段的开始，用于定义已初始化的全局或静态变量。`.word` 初始化一个变量。`.text` 表示代码段的开始。
2. 34 这是第九个斐波那契数
3.  t3
## Exercise 2

1. `t0`
2. `s0`
3. `s2`
4. `loop` 下面部分的代码
5. 使用 `sp` 寄存器

## Exercise 3

本题需要我们使用 riscv 语法完成阶乘函数，题目在 `main` 函数中已经告诉我们将 `a0` 作为参数 `n ` 传递给函数，我们只需要完成函数内部实现即可。

需要注意的是，根据**Calling Convention**，`factorial` 作为 callee，是要在其函数内部分配栈空间的，这样才能将通过函数调用后的参数返回给 `main` 函数

```asm
factorial:
    addi sp, sp, -4
    sw s0, 0(sp)

    addi t0, x0, 1 # int i = 1
    addi s0, x0, 1 # int res = 1
    loop:
        blt a0, t0, exit # 若 i > n 则跳转到结尾
        mul s0, s0, t0
        addi t0, t0, 1
        j loop
    exit:
        mv a0, s0

        lw s0, 0(sp)
        addi sp, sp, 4
        ret
```

## Exercise 4

本题需要我们修改若干个关于**Calling Convention**的问题，其中大部分都是关于**栈内存**的问题，总体来说并不难，只需要注意哪个寄存器是要在被调用后还需要保存的，就将这个寄存器分配一块属于它的栈空间即可。

需要注意的是，其中 `inc_arr` 函数中还调用了一个函数，在这里还需要分配一块空间。

```asm
simple_fn:
    mv a0, x0
    li a0, 1
    ret
    
naive_pow:
    # BEGIN PROLOGUE
    addi sp, sp, -4
    sw s0, 0(sp)
    # END PROLOGUE
    li s0, 1
naive_pow_loop:
    beq a1, zero, naive_pow_end
    mul s0, s0, a0
    addi a1, a1, -1
    j naive_pow_loop
naive_pow_end:
    mv a0, s0
    # BEGIN EPILOGUE
    lw s0, 0(sp)
    addi sp, sp, 4
    # END EPILOGUE
    ret
    
inc_arr:
    # BEGIN PROLOGUE
    #
    # FIXME What other registers need to be saved?
    #
    addi sp, sp, -16
    sw ra, 0(sp)
    sw s0, 4(sp)
    sw s1, 8(sp)
    # END PROLOGUE
    mv s0, a0 # Copy start of array to saved register
    mv s1, a1 # Copy length of array to saved register
    li t0, 0 # Initialize counter to 0
inc_arr_loop:
    beq t0, s1, inc_arr_end
    slli t1, t0, 2 # Convert array index to byte offset
    add a0, s0, t1 # Add offset to start of array
    # Prepare to call helper_fn
    #
    # FIXME Add code to preserve the value in t0 before we call helper_fn
    # Hint: What does the "t" in "t0" stand for?
    # Also ask yourself this: why don't we need to preserve t1?
    #
    sw t0, 12(sp)
    jal helper_fn
    lw t0, 12(sp)
    # Finished call for helper_fn
    addi t0, t0, 1 # Increment counter
    j inc_arr_loop
inc_arr_end:
    # BEGIN EPILOGUE
    lw ra, 0(sp)
    lw s0, 4(sp)
    lw s1, 8(sp)
    addi sp, sp, 16
    # END EPILOGUE
    ret
    
helper_fn:
    # BEGIN PROLOGUE
    addi sp, sp, -4
    sw s0, 0(sp)
    # END PROLOGUE
    lw t1, 0(a0)
    addi s0, t1, 1
    sw s0, 0(a0)
    # BEGIN EPILOGUE
    lw s0, 0(sp)
    addi sp, sp, 4
    # END EPILOGUE
    ret
```

## Exercise 5

本题需要我们完成一个 `map` 函数，其中参数 1 给定一个列表，参数 2 给定一个函数。先打印原列表，再打印经过函数处理后的列表。

题目要求在注释里面都给定了，讲的也很清楚。

```asm
# load the address of the function in question into a1 (check out la on the green sheet)
la a1, square

map:
    # Prologue: Make space on the stack and back-up registers
    addi sp, sp -12
    sw ra, 0(sp)
    sw s0, 4(sp)
    sw s1, 8(sp)

    beq a0, x0, done    # If we were given a null pointer (address 0), we're done.

    add s0, a0, x0  # Save address of this node in s0
    add s1, a1, x0  # Save address of function in s1

    # Remember that each node is 8 bytes long: 4 for the value followed by 4 for the pointer to next.
    # What does this tell you about how you access the value and how you access the pointer to next?

    # load the value of the current node into a0
    # THINK: why a0?
    lw a0, 0(s0)

    # Call the function in question on that value. DO NOT use a label (be prepared to answer why).
    # What function? Recall the parameters of "map"
    jalr s1

    # store the returned value back into the node
    # Where can you assume the returned value is?
    sw a0, 0(s0)

    # Load the address of the next node into a0
    # The Address of the next node is an attribute of the current node.
    # Think about how structs are organized in memory.
    lw a0, 4(s0)

    # Put the address of the function back into a1 to prepare for the recursion
    # THINK: why a1? What about a0?
    mv a1, s1

    # recurse
    jal ra, map

done:
    # Epilogue: Restore register values and free space from the stack
	lw s1, 8(sp)
	lw s0, 4(sp)
	lw ra, 0(sp)
	addi sp, sp, 12	
    addi sp, sp, 12

    jr ra # Return to caller
```

# Lab 04

## Exercise 1

本题需要我们找出代码中的错误，具体错误如下

1. 在加载字的时候应使用 `lw`
```asm
	lw t1, 0(s0) # add t1, s0, x0      # load the address of the array of 	current node into t1
```

2. 对于一个数组来说，每到下一个数组内容，需要增加 4 字节，故要注意偏移量的问题
```asm
	addi t3, x0, 4
    mul t3, t3, t0
    add t1, t1, t3      # offset the array address by the count
    lw a0, 0(t1)        # load the value at that address into a0
```

3. 每当我们调用一个函数时，要先对这个返回值分配内存，随后再释放内存
```asm
addi sp, sp -4
sw t1, 0(sp)
jalr s1             # call the function on that value.
lw t1, 0(sp)
addi sp, sp 4
```

4. 加载数组的值到寄存器中应使用 `lw`
```asm
    lw a0, 8(s0)        # load the address of the next node into a0
```

5. 递归调用函数时
```asm
    mv a1, s1 # lw a1, 0(s1)        # put the address of the function back into a1 to prepare for the recursion
```

## Exercise 2

本题向我们介绍了**如何使用动态地址加载一个字**

动态索引通常需要我们将索引转换为字节偏移量，也就是乘 4

```asm
f:
    addi t0, a0, 3 # 从-3到3，+3后变为0到6 求index
    slli t0, t0, 2 # 索引转换为字节偏移量 t0 = index * 4 slli左移两位代表 *4
    add a1, a1, t0 # 计算元素地址，a1 = arr + (index * 4)
    lw a0, 0(t0) # 加载 arr[index] 到 a0

    jr ra               # Always remember to jr ra after your function!
```

