---
layout: post
title:  "Block 结构体中 forwarding 成员变量的作用"
date:   2016-9-1 18:30:00
---

## 前言

本文假设读者已经了解了 Objective-C 语言中 Blcok 的基础原理知识，如果你还没有了解到这些知识，建议访问这个网址来学习：[blog.devtang.com](http://blog.devtang.com/2013/07/28/a-look-inside-blocks/)


## 回顾

我们知道，在 Block 内部是不可以改写截获的局部变量的，如果非要改写，就要使用 `__block` 存储域修饰符来修饰这个变量，就像下面这个样子：

```
void foo_(){
    __block int i = 2;
    long (^myBlock)(void) = ^long() {
    		i = 5;
          return i * 3;
    };
    long r = myBlock();
    printf("%ld", r);
}
```

我们通过 clang 改写上面的代码后了解到，使用 __block 修饰变量后，代码量变得很大，并且经过 `__block` 修饰的变量其实已然转化成了对象来对待：

```
struct __Block_byref_i_0 {
  void *__isa;
__Block_byref_i_0 *__forwarding;
 int __flags;
 int __size;
 int i;
};
```
上面的结构体组成中，最后的成员变量 `i` 的值即为用来我们原始声明变量的值，我们也了解到接下来程序中访问这个原始的变量的地方都被替换成为 `i->__forwarding->i`, 其中我们注意到 `__forwarding` 指针变量指向的是跟该结构体类型一致的值，我们可以粗暴的理解为这个变量指向了结构体自己，这确实引起了我们的注意，为什么要搞一个变量指向自己，为什么不直接使用 `i->i` 来访问我们原始的变量的值呢？

接下来我们就一步一步解开这个神秘变量的面纱!

## 原理探究

从作用域上来讲，我们知道 Blcok 可以分为栈上的和堆上的。当栈上创建的 Block 超出作用域后依然要访问它，就需要把该 Block 拷贝到堆上，同时也需要把该 Block 持有的栈上的对象也拷贝到堆上。也就是说如果一个 Block 持有一个栈上的 `__Block_byref_i_0` 的对象的话，经过拷贝，堆上也会有一个 `__Block_byref_i_0` 对象，这样，在栈上和堆上就同时有了两份一样的对象（当然，只是对象的值是相等的），怎么保证这两份对象之间数据的统一，是个很关键的问题！

我们从源代码入手，将 Blcok 持有的 `__Block_byref_i_0 ` 对象拷贝到堆上是通过下面这个函数来完成的：

```
 _Block_object_assign((void*)&dst->i, (void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/ 
```

我们注意到这个函数的最后一个参数是个标记要拷贝的这个变量的类型，这里的 8 代表 `BLOCK_FIELD_IS_BYREF` 是说这个变量是 `__block` 修饰的变量。这个函数的后续动作我们可以通过访问苹果开源的 Block 源代码来查看，或者直接访问这个网址来在线浏览：[opensource.apple.com](http://opensource.apple.com/source/libclosure/libclosure-63/runtime.c)

源代码中的判断比较多，我直接整理出了针对 `__block` 修饰的变量的拷贝操作，大概做了下面的事情：

```
static void _Block_byref_assign_copy(void *dest, const void *arg, const int flags) {
    struct Block_byref **destp = (struct Block_byref **)dest;
    struct Block_byref *src = (struct Block_byref *)arg;
    // src points to stack
    struct Block_byref *copy = (struct Block_byref *)_Block_allocator(src->size, false, isWeak);
    copy->forwarding = copy; // patch heap copy to point to itself (skip write-barrier)
    src->forwarding = copy;  // patch stack to point to heap copy
    copy->size = src->size;
    // assign byref data block pointer into new Block
    _Block_assign(src->forwarding, (void **)destp);
}
```

从上面的代码中可以看到，这个函数核心的动作就是在堆上新创建一个对象 `copy`, 将 `copy` 变量的 `forwarding` 指针指向堆上的 `copy` 自己，然后再将栈上的原始对象的 `forwarding` 也指向堆上的 `copy` 变量。

现在回过头再看这个 `forwarding` 这个变量，我们终于明白它的意义：保证无论在栈上还是堆上，我们通过 `forwarding` 访问到的始终是同一个对象！

## 总结
通过研究苹果的源代码我们进一步了解了 Block 的工作原理，这样我们在工作中就对是否使用 `__block` 修饰变量以及修饰变量后实质对代码产生的影响都有了很好的把握，有助于我们写出更完善的代码。





