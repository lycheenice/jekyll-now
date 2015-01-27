---
layout: post
title: 向量化、SIMD架构：你应该知道些什么
---

*本文为译文，原文链接[在此](http://goparallel.sourceforge.net/vectorization-simd-architecture-need-know/)*

>**并行化编程的一种形式是向量化，这种形式允许用单条指令实现多个数据的操作。在这一系列的第一篇文章里，Jeff Cogswell将带你走进Intel处理器的向量化世界。**

向量化是我们在[Go Parallel](http://goparallel.sourceforge.net/)中会大量涉及的题目。但是，为了新的读者能够理解，我们还是首先普及下什么是向量化以及Intel C++编译器是如何实现向量化的。

为确保理解向量化，您首先应该知道一点**处理器架构**和**汇编语言**的知识：

######Intel Math Architecture
如大多数处理器一样，Intel处理器中包含了大量的寄存器，这些寄存器本质上来说是内建的存储器。如果你想进行两个整数的相加，你需要将两个数存放在独立的寄存器中，然后使用汇编指令`ADD`进行操作。指令完成后，结果会存放在提前指定的一个或两个寄存器中，代替之前存放的数据。

当然你还有其它选择。例如，你可以将操作数存放在内存而不是寄存器中，或者你可以直接使用立即数和寄存器中存放的内容相加。下面是一些实际的例子。

```asm
ADD R8L, AL  ;将两个寄存器R8L、AL中的数相加，结果存放到R8L中
ADD EAX, 20  ;将寄存器EAX和立即数20相加，结果存放到EAX中
```

![8086](http://upload.wikimedia.org/wikipedia/commons/d/d2/KL_USSR_KP1810BM86.jpg)

这些功能可以追溯到早年的Intel处理器，包括最早的8086处理器，这颗芯片于1978年问世，当时最早的IBM个人电脑及兼容机都采用了8086处理器。当时这颗芯片还不具备浮点运算功能。正因为如此，Intel为它增添了浮点协处理器完成浮点操作。直到1989年推出的i486才将独立的浮点处理器集成到一颗芯片上。

![i486](http://upload.wikimedia.org/wikipedia/commons/7/77/Intel_i486_dx4_100mhz_2007_03_27.jpg)

从那时起，处理器开始采用内建的**浮点单元**处理浮点数学运算。在执行浮点操作前，参与运算的操作数需要提前放置在寄存器中，然后通过汇编指令发起浮点运算。参与浮点运算的是专用寄存器，这些寄存器要比单个浮点数本身大的多。

Intel处理器提供三种类型的浮点数。这些浮点数的长度分别为4bytes、8bytes、10bytes，它们依次被命名为**单精度浮点数、双精度浮点数、扩展精度浮点数**。

![float/double](http://www.ibm.com/developerworks/cn/java/j-jtp0114/float.gif)

在C++语言中，`float`类型被定义为**单精度浮点数**，`double`类型被定义为**双精度浮点数**。10bytes扩展精度浮点类型并没有被广泛使用，许多高级语言甚至没有对此支持。在C++编译器中以`long-double`的形式提供支持。（但如果仔细研究该类型的大小，你会发现`long-double`实际为16bytes。编译器分配了比原始类型更大的空间去存储数据结构，为的是地址对齐。）

转而，寄存器的尺寸也越来越大。我们常以bits为单位衡量寄存器。在九十年代中期，**Pentium**处理器拥有的寄存器大小为64bits。这意味着你可以将一个完整的双精度浮点数或两个单精度浮点数放入一个寄存器。下面到了*见证奇迹的时刻*。

![simd](http://origin.arstechnica.com/cpu/1q00/simd/figure6.gif)

如果你在一个寄存器中存放了两个单精度浮点数，便可以用一条指令同时完成两个浮点运算操作。这就叫做**向量化**。**向量话化**是并行计算的一种，它可以提升程序性能。**向量化**的另一个名字是**SIMD**（发音为*sim-dee*）,全称是**Single Istruction, Multiple Data**。Intel公司将SIMD技术应用于自家处理器并命名为**MMX**，这个技术在P5处理器中开始得到应用。

![mmx](http://www.vector-logo.net/logo_preview/ai/i/Intel_MMX_big_logo.png)

在过去的20年中，SIMD寄存器的数量和大小在新一代处理器上快速增长。于1999年推出的**Pentium III**，拥有8个128bits（16bytes）大小的SIMD寄存器。意味着单个寄存器中可以存放4个单精度浮点数或者2个双精度浮点数。2011年推出的**Sany Bridge**架构，支持**AVX**技术（Advanced Vector Extensions）,该技术使用256bites大小的SIMD寄存器——支持8单精度或4双精度的浮点数。在2015年，我们很有可能看到支持512bits的AVX-512技术应用于Intel最新生产线。

![avx](http://cdn.wccftech.com/wp-content/uploads/2014/09/Intel-Xeon-E5-2600-V3-Haswell-EP-AVX-2-Instruction-Set.jpg)

#####结束语

通过利用宽向量寄存器，我们可以在一条指令里实现多个操作数的计算，带来性能的大幅提升。下一讲中，我们将看到向量操作的更多细节，以及如何用汇编语言实现向量操作。然后，我们将探索如何在C++中实现向量化操作，编译器又是自动实现的。

#####本系列其他博客
>[1] [Vectorization: Array Management Made Easier, Vectors vs. Scalar: Moving Packed Data](http://goparallel.sourceforge.net/vectorization-array-management-made-easier/)

>[2] [Writing Vectorized Code with C++](http://goparallel.sourceforge.net/writing-vectorized-code-c/)
