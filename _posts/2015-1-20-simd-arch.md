---
layout: post
title: 向量化、SIMD架构：你应该知道些什么
---
*本文为翻译文章，原文链接为：[http://goparallel.sourceforge.net/vectorization-simd-architecture-need-know/](http://goparallel.sourceforge.net/vectorization-simd-architecture-need-know/)*

**并行化编程的一种形式是向量化，这种形式允许用单条指令实现多个数据的操作。在这一系列的第一篇文章里，Jeff Cogswell将带你走进Intel处理器的向量化世界。**

向量化是我们在[Go Parallel](http://goparallel.sourceforge.net/)中会大量涉及的题目。但是，为了新的读者能够理解，我们还是首先普及下什么是向量化以及Intel C++编译器是如何实现向量化的。

为确保理解向量化，您首先应该知道一点**处理器架构**和**汇编语言**的知识：

###Intel Math Architecture
如大多数处理器一样，Intel处理器中包含了大量的寄存器，这些寄存器本质上来说是内建的存储器。如果你想进行两个整数的相加，你需要将两个数存放在独立的寄存器中，然后使用汇编指令`ADD`进行操作。指令完成后，结果会存放在提前指定的一个或两个寄存器中，代替之前存放的数据。

当然你还有其它选择。例如，你可以将操作数存放在内存而不是寄存器中，或者你可以直接使用立即数和寄存器中存放的内容相加。下面是一些实际的例子。

```asm
ADD R8L, AL  ;将两个寄存器R8L、AL中的数相加，结果存放到R8L中
ADD EAX, 20  ;将寄存器EAX和立即数20相加，结果存放到EAX中
```
C语言代码演示
```c++
#include <stdlib.h>
#define MAX = 100;

int main(int argc, char **argv){
  int val1 = atoi(argv[1]);
  val1 = val1 * 2;
  printf("I will get %d %s",val1%MAX, argv[2]);
  return 0;
}
```
