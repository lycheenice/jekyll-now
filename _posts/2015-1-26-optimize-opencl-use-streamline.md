---
layout: post
title: 在Mali GPU上使用DS-5 Streamline优化复杂的OpenCL程序
---
*本文为翻译文章，原文地址[在此](http://community.arm.com/groups/arm-mali-graphics/blog/2015/01/23/using-ds-5-streamline-to-optimize-complex-opencltm-applications-on-mali-gpus)，原文作者为[Tim Hartley](http://community.arm.com/people/timhar01)*

**异构应用**——那些在多核处理器，例如CPU和GPU上同时运行的代码——始终是非常难以优化的。你不只需要考虑不同的代码段在不同的设备上的最佳运行效果，而且需要考虑它们之间的交互开销如何。是一个处理器在等待另一个处理器？是否进行了一次不必要的图像拷贝？你利用了多少GPU资源？程序的瓶颈在哪里？所有的这一切都不能凭空判定。

性能分析工具可以给出以上答案，至少是一部分答案。**DS-5 Streamline 性能分析工具**是ARM提供的性能分析工具，可以观察到OpenCL程序的一些有趣特性。**Streamline**是**ARM DS-5 Development Studio**中的一个组件，是ARM处理器开发过程中端到端工具集。

本文着眼于DS-5 Streamline、复杂异构应用以及如何对它进行优化。在这偏博客中，我会初步介绍DS-5工具以及如何利用它对OpenCL程序进行优化。

###DS-5简介

![DS-5](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12121/pic1.jpg)

DS-5 Streamline可以连接真实设备并实时获取硬件counter信息。那些你选择的counter会实时绘制在时间轴中，在trace流中可以同时包括CPU和GPU的counter信息。以上面的图像为例，以时间轴的方式展示了trace中counter的数据信息。在图像的最顶端，绿色的是双核CPU的负载情况，紧随其下的是蓝色的GPU图形活动信息和红色的GPU计算活动信息。接下来是其他硬件counter的信息以及其他系统trace信息。

在时间轴的基础上，你可以进一步钻取*(drill down)*CPU端的进程信息，然后根据系统调用下面的各类信息分析程序性能。如过伴随Mali系列GPU，你还可以通过制定硬件counter并将其绘制与CPU信息旁边。这项功能允许你同时分析GPU图像处理任务和OpenCL计算任务，支持组件和计算核心上的细节分析。最新加入的功能中，OpenCL时间轴让单独的kernel和kernel链分析成为了可能。

###优化流程

上面介绍了很多Streamline的基本概念，那么典型的OpenCL优化流程应该是怎样的呢？

一般情况下，如果我们想要实现基于CPU和GPU的解决方案，首先应该搭建CPU-only的软件版本。在这个版本中你会找出代码中需要优化的**捣蛋鬼**。接下来，根据一些黄金参考准则以及目前实现的效果，你可以粗略估算出在多核平台上优化该**捣蛋鬼**可以获得的预期收益。

下一步通常是进行初步移植*(naive port)*，在这个版本中将完成从CPU到GPU的功能性的移植。在这个粗糙的版本上不要对性能的提升抱有太大的期望，甚至性能下降都是可接受的。在这个阶段，重要的是建立异构编程模型。

接下来，你需要考虑如何进行优化了。对初步移植的版本进行性能剖析往往是必不可少的。它可以帮助你找出程序的热点，在热点上集中精力进行优化，往往可以达到事半功倍的效果。这一步的分析往往暗示了对算法进一步优化的重要线索。

为了充分挖掘硬件的计算资源，对目标硬件架构的理解是必不可少的。下面，让我们先介绍一点Mali GPU的背景知识。

###Mali GPUs的OpenCL执行模型

首先，然我们看一下OpenCL执行模型是如何映射到Mali GPUs上的。

![excute model](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12103/pic2.jpg)

OpenCL中的work-item映射为shader core上的线程*(thread)*，每一个硬件线程拥有自己的寄存器组、PC、SP以及私有堆栈。每个shader core可以同时运行多达256个硬件线程，每个线程都可以进行原生的向量计算。

OpenCL中的work groups——work items的集合——被映射到一个shader core之上。Workgorup可以拥有barriers、local atomics以及local memory cache（在硬件上拥有同global memory一样的实现，在Mali GPU上使用local memory是出于方便而不是性能）。

ND Range——OpenCL kernel的全部工作负载，负责划分workgroups并将它们分配到可用的shader core上。支持global atomics，拥有global memory cache。

正如我们看到的，相比于其他GPU架构，Mali GPU cores做为相对复杂的设备可以同时运行上百个线程。

###Mali GPU Core

接下来，让我们看看Mali GPU core的年内部：

![Mali core](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12107/pic3.jpg)

每个GPU核心包含，两条ALU流水线，一条L/S流水线以及一条Texture流水线。线程从上部进入流水线，







####未完待续 >>> 

