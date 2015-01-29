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

当问题集中在特定应用的时候，DS-5 Streamline往往可以很快的显示出来。下一张图片展示了计算机视觉应用运行在CPU和OpenCL GPU上的统计信息。前几秒种，程序还在正常运行，然后，突然速度下降，图像的帧率下降到之前的一半。

![streamline-4](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12112/crop3.jpg)

我们看到，Trace信息捕捉到了这一幕。在时间轴标记的左边，CPU和GPU相对高效的运行。突然，图形边长，我们可以看到在GPU和CPU高负载运行的间隔明显变大。图像上端显示在绿色柱状图下的红色部分表示系统调用消耗的CPU资源。这些trace结果很好的反映了应用的原始问题并且可以很好的帮助我们理解视频的处理流程。

掌握系统运行的全局信息的好处是，我们可以得到应用在多核处理器的核心指尖以及异构处理器指尖的性能情况，这在本例中是非常有用的。

![streamline-5](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12113/crop4.jpg)

这里我们向下滑动右侧滚动条以展示更多的信息——特别是Mali核心中的其他Activities信息。你可以看到很多行的counter信息，需要特别注意的有arithmetic、load-store和texture pipes和其它的一些cache hits，misses信息。在这些统计图形上悬停可以显示具体的计数器数值。

![streamline-6](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12114/crop5.jpg)

举例来说，在Mali Load/Stroe Pipe这一行统计信息中，上面较亮的绿色图形表示指令发射数，下面较暗的绿色曲线表示指令的完成数。这条时间线统计了L/S指令发射和完成的数目，这些数据实际上反应了此刻的访存效率。

下面的trace信息来自同一个应用，这幅图片在稍早之前出现过，在这里我们利用OpenCL filter工具再次观察它。

![streamline-7](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12115/crop6.jpg)

如果近距离的观察，我们可以看到应用运行的效率如何。这里我们放大了CPU trace信息——截图上部绿色的柱状图——来显示该平台两个CPU核心的使用情况。需要注意的是，蓝色表示的是GPU部分的信息，红色表示GPU中图像滤波处理。

![streamline-8](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12116/mag.jpg)

下面，我们来观察一个完成的单帧周期：
1. 首先，CPU进入高负载状态，引导下面的计算工作。
2. 紧接着，GPU进入活跃状态，开始计算工作，CPU进入相对空闲期。
3. 随着图像滤波的完成，CPU又迎来少量的工作任务，设置图像渲染所需参数。
4. 最后，GPU进行该帧图像渲染工作，直到序列中的下一帧图像到来

So in a snapshot we have this holistic and heterogeneous overview of the application and how it is running.  Clearly we could aim for much better performance here by pipelining the workload to avoid the idle gaps we see.  There is no reason why the CPU and GPU couldn’t be made to run more efficiently in parallel, and this trace shows that clearly.

###OpenCL Timeline
There are many features of DS-5 Streamline, and I’m not going to attempt to go into them all.  But there’s one in particular I’d like to show you that links the latest Mali GPU driver release to the latest version of DS-5 (v5.20), and that’s the OpenCL Timeline.

![opencl timeline](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12122/pic1.jpg)

In this image we’ve just enabled the feature – it’s the horizontal area at the bottom.  This shows the running of individual OpenCL kernels, the time they take to run, any overhead of sync-points between CPU and GPU etc.

![opencl details](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12118/crop7.jpg)

Here we have the name of each kernel being run along with the supporting host-side setup processes   If we hover over any part of this timeline…

![exqueue](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12119/crop8.jpg)

… we can see details about the individual time taken for that kernel or operation.  In terms of knowing how then to target optimizations, this is invaluable.
 
Here’s another view of the same feature.

![exqueue 2](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12120/pic15.jpg)

We can click the “Show all dependencies” button and Streamline will show us visually how the kernels are interrelated.  Again, this is all within the timeline, fitting right in with this holistic view of the system.  Being able to do this – particularly for complex, multi-kernel OpenCL applications is becoming a highly valuable tool for developers in helping to understand and improve the performance of ever-more demanding applications.

###Optimizing Memory Accesses

So once you have these hardware counters, what sort of use should you make of them?
 
Generally speaking, the first thing to focus on is the use of memories. The SoC only has one programmer controlled memory in the system – in other words, there is no local memory, it’s all just global.  The CPU and GPU have the same visibility of this memory and often they’ll have a shared memory bus.  Any overlap with memory accesses therefore might cause problems.
 
If we want to shift back and forth between CPU and GPU, we don’t need to copy memory (as you might do on a desktop architecture).  Instead, we only need to do cache flushes.  These can also take time and needs minimising. So we can take an overview with Streamline of the program allowing us to see when the CPU was running and when the GPU was running, in a similar way to some of the timelines we saw earlier.  We may want to optimize our synchronisation points so that the GPU or CPU are not waiting any longer than they need to. Streamline is very good at visualising this.

###Optimizing GPU ALU Load

With memory accesses optimized, the next stage is to look more closely at the execution of your kernels.  As we’ve seen, using Streamline we can zoom into the execution of a kernel and determine what the individual pipelines are doing, and in particular determine which pipeline is the limiting factor.  The Holy Grail here – a measure of peak optimization – is for the limiting pipe to be issuing instructions every cycle.
 
I mentioned earlier that we have a latency-tolerant architecture because we expect to have a great many threads in the system at any one time. Pressure on register usage, however, will limit the number of threads that can be active at a time.  And this can introduce latency issues once the number of threads falls sufficiently.  This is because if there are too many registers per thread, there are not enough registers for as many threads in total.  This manifests itself in there being too few instructions being issued in the limiting pipe.  And if we’re using too many registers there will be spilling of values back to main memory, so we’ll see additional load/store operations as a result.  The compiler manages all this, but there can be performance implications of doing so.
 
An excessive register usage will also result in a reduction in the maximum local workgroup size we can use.
 
The solution is to use fewer registers.  We can use smaller types – if possible.  So switching from 32 bit to 16 bit if that is feasible.  Or we can split the kernel into multiple kernels, each with a reduced number of registers.  We have seen very large kernels which have performed poorly, but when split into 2 or more have then overall performed much better because each individual kernel needs a smaller number of registers.  This allows more threads at the same time, and consequently more tolerance to latency.

###Optimizing Cache Usage

Finally, we look at cache usage.  If this is working badly we would see many L/S instructions spinning around the L/S pipe waiting for the data they have requested to be returned. This involves re-issuing instructions until the data is available.  There are GPU hardware counters that show just what we need, and DS-5 can expose them for us.
 
This has only been a brief look at the world of compute optimization with Mali GPUs.  There’s a lot more out there.  To get you going I’ve included some links below to malideveloper.arm.com for all sorts of useful guides, developer videos, papers and more.

###延伸内容
> - Download DS-5 Streamline: [ARM DS-5 Streamline - Mali Developer Center Mali Developer Center](http://malideveloper.arm.com/develop-for-mali/tools/software-tools/arm-development-studio-5/)

> - Mali-T600 Series GPU OpenCL Developer Guide: [Mali-T600 Series GPU OpenCL Developer Guide - Mali Developer Center Mali Developer Center](http://malideveloper.arm.com/develop-for-mali/tutorials-developer-guides/developer-guides/mali-t600-series-gpu-opencl-developer-guide/)

> - GPU Compute, OpenCL and RenderScript [Tutorials](http://malideveloper.arm.com/develop-for-mali/opencl-renderscript-tutorials/)

####未完待续 >>> 

