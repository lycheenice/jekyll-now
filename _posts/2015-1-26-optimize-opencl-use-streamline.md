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

接下来，让我们看看Mali GPU core的内部：

![Mali core](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12107/pic3.jpg)

每个GPU核心包含，两条ALU流水线，一条L/S流水线以及一条Texture流水线。线程从上部进入并被分配到这三类管线之一，穿过流水线，循环至头部运行下一条指令，直到整个线程执行完成，从底部推出流水线。在一般情况下，同时有非常多的线程遵循上述路径运行，不同线程的不同指令同时充斥着流水线。

####Load/Store

让我们想想一下，第一条指令是一条Load指令。它进入循环并在L/S管线中运行。如果数据是可访问的，这条线程就可以进如下一条指令的循环。如果数据还没有到达，这条指令会进行等待，直到获取数据。

####ALUs

想象下一条是计算指令。这条指令会进入计算管线。Mali Midgard ALU支持SIMD运算。指令格式为VLIW——very long instruction word——支持在同一条指令中进行多种操作。举例来说，可以在一条指令中同时包含：一个向量加操作、一个向量成操作以及几个标量操作。这种指令导致某些操作可以是免费的*(apperaing as free)*，因为计算单元可以同时并行的执行一条指令中的多个操作。最终，很多内建函数可以通过硬件加速达到更快的执行速度*(BIFL-Build In Function Library)*。

这颗ALU及复杂和强大与一身，被设计为同时多线程，并以此隐藏延迟。优化从根本上来讲就是隐藏延时。只要有其它可运行的线程，我们就不必担心单独的线程中因为某些等待数据而产生的访存延迟。

每一条流水线之间互相独立，同样的，每一个线程之间也互相独立。程序运行的总时间取决于每个进程需要最多周期的那条流水线。假设L/S操作占据主要地位，那么L/S流水线就会成为限制因素。所以，为了有效的优化程序，我们必须首先找出哪条管线是限制因素。

###硬件计数器

为了得到上述信息，我们需要借助GPU的硬件计数器信息。这些信息可以指出这些核心的哪些部分执行了哪些工作。继而，帮助我们确定性能瓶颈。

通过Streamline我们可以得到大量的counter信息。一些counter是针对于单个核心的，可以让你观察并理解管线内部的工作情况。一些counter是统计全局信息的，例如active cycles数量。

我们可以利用DS-5访问这些counters信息，下面让我们来看一些Streamline的工作截图。

![streamline-1](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12108/pic4.jpg)

这是一幅完整的系统截图。最上面一行的绿色柱状图描述了CPU activity，下一行的蓝色柱状图为GPU activity，红色柱状图为GPU中专用计算管线的activity。

![streamline-2](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12110/crop.jpg)

我们可以采用各种手段去定制这些曲线。截图中显示的并不是最细节的信息，你可以根据自己需要统计的信息去选择观察窗口的范围。根据需要，Streamline支持查看特定线程的CPU、GPU信息。

![streamline-3](http://community.arm.com/servlet/JiveServlet/downloadImage/38-4357-12111/907-373/crop2.jpg)

在这张图片上，你可以看到有关L2 Cache的统计信息，它由蓝色的曲线表示。在最下方还能看到计算管线的信息。我们可以向下滚动观察更多的信息或者通过放大看到更多的细节。

当问题集中在特定应用的时候，DS-5 Streamline往往可以很快的显示出来。下一张图片展示了计算机视觉应用运行在CPU和OpenCL GPU上的统计信息。  It would run fine for a number of seconds, and then seemingly randomly would suddenly slow down significantly, with the processing framerate dropping in half。

![streamline-4](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12112/crop3.jpg)

You can see the trace has captured the moment this slowdown happened. To the left of the timeline marker we can see the CPU and GPU working reasonably efficiently.  Then this suddenly lengthens out, we see a much bigger gap between the pockets of GPU work, and the CPU activity has grown significantly.  The red bars in amongst the green bars at the top represent increased system activity on the platform.  This trace and others like it were invaluable in showing that the initial problem with this application lay with how it was streaming and processing video.
 
One of the benefits of having the whole system on view is that we get a holistic picture of the performance of the application across multiple processors and processor types, and this was particularly useful in this example.

![streamline-5](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12113/crop4.jpg)

Here we’ve scrolled down the available counters in the timeline to show some others – in particular the various activities within the Mali GPU’s cores.  You can see counter lines for a number of things, but in particular the arithmetic, load-store and texture pipes – along with cache hits, misses etc.  Hovering over any of these graphs at any point in the timeline will show actual counter numbers.

![streamline-6](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12114/crop5.jpg)

Here for example we can see the load/store pipe instruction issues at the top, and actual instructions on the bottom.  The difference in this case is a measure of the load/store re-issues necessary at this point in the timeline – in itself a measure of efficiency of memory accesses.  What we are seeing at this point represents a reasonably healthy position in this regard.
 
The next trace is from the same application we were looking at a little earlier, but this time with a more complex OpenCL filter chain enabled.

![streamline-7](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12115/crop6.jpg)

If we look a little closer we can see how efficiently the application is running.  We’ve expanded the CPU trace – the green bars at the top – to show both the cores we had on this platform.  Remember the graphics elements are the blue bars, with the image processing filters represented by the red.

![streamline-8](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12116/mag.jpg)

Looking at the cycle the application is going through for each frame:
 
1. Firstly there is CPU activity leading up to the compute job.
2. Whilst the compute job then runs, the CPU is more or less idle.
3. With the completion of the compute filters, the CPU does a small amount of processing, setting up the graphics render.
4. The graphics job then runs, rendering the frame before the sequence starts again.
 
So in a snapshot we have this holistic and heterogeneous overview of the application and how it is running.  Clearly we could aim for much better performance here by pipelining the workload to avoid the idle gaps we see.  There is no reason why the CPU and GPU couldn’t be made to run more efficiently in parallel, and this trace shows that clearly.




####未完待续 >>> 

