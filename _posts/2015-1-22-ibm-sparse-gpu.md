---
layout: post
title: 在GPU平台上基于编译时和运行时策略的稀疏矩阵向量乘优化
---

*本文为翻译文章*
*[原文链接](http://www.idi.ntnu.no/~elster/master-studs/spampinato/ibm-sparse-gpu.pdf)*

##在GPU平台上基于编译时和运行时策略的稀疏矩阵向量乘优化##

###摘要###

我们亲眼目睹了GPU作为大规模并行运算处理器所取得的优异成绩。而且，近些年通用并行编程API的诞生更加方便了GPU通用编程。这些API包括Nvidia的**CUDA**，AMD的**Stream**，以及**OpenCL**。**稀疏矩阵向量乘** *(Sparse Matrix-Vector muliplication / SpMV)* 是科学计算中十分重要而且耗时的核心算子*(kernel)*。间接和不规律的访存特征导致了SpMV每次浮点操作都有更多的访存开销，这为在所有架构上的优化都带来了很大挑战。

在这片文章里，我们评估了在NVIDIA GPUs硬件和CUDA编程模型下实现高性能SpMV算子在编译时*(compile-time)*和运行时*(run-time)*两大方面s所遇到的各类优化挑战。

**编译时**优化包括：
>1. 挖掘非同步并行性
2. 基于*affinity towards optimal memory access pattern*优化线程映射
3. 优化off-chip访存操作减小访存延迟
4. 挖掘数据重用性

**编译时**优化包括：

**未完待续>>>**
