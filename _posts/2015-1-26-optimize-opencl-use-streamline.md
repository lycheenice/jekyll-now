---
layout: post
title: 在Mali GPU上使用DS-5 Streamline优化复杂的OpenCL程序
---
本文为译文，原文地址[在此](http://community.arm.com/groups/arm-mali-graphics/blog/2015/01/23/using-ds-5-streamline-to-optimize-complex-opencltm-applications-on-mali-gpus),原文作者为[Tim Hartley](http://community.arm.com/people/timhar01)

**异构应用**——那些在多核处理器，例如CPU和GPU上同时运行的代码——始终是非常难以优化的。你不只需要考虑不同的代码段在不同的设备上的最佳运行效果，而且需要考虑它们之间的交互开销如何。是一个处理器在等待另一个处理器？是否进行了一次不必要的图像拷贝？你利用了多少GPU资源？程序的瓶颈在哪里？所有的这一切都不能凭空判定。

性能分析工具可以给出以上答案，至少是一部分答案。**DS-5 Streamline 性能分析工具**是ARM提供的性能分析工具，可以观察到OpenCL程序的一些有趣特性。**Streamline**是**ARM DS-5 Development Studio**中的一个组件，是ARM处理器开发过程中端到端工具集。

本文着眼于DS-5 Streamline、复杂异构应用以及如何对它进行优化。在这偏博客中，我会初步介绍DS-5工具以及如何利用它对OpenCL程序进行优化。

###DS-5简介

![DS-5](http://community.arm.com/servlet/JiveServlet/showImage/38-4357-12121/pic1.jpg)

DS-5 Streamline可以连接真实设备并实时获取硬件counter信息。那些你选择的counter会实时绘制在时间轴中，在trace流中可以同时包括CPU和GPU的counter信息。以上面的图像为例，以时间轴的方式展示了trace中counter的数据信息。在图像的最顶端，是双核CPU的负载情况，






####未完待续 >>> 

