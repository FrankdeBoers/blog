---
title: Camera OpenGL FBO的理解
tags: Android OpenGL Camera
categories: Android
---
本文主要讲解OpenGL FBO的概念。


# 背景概念：

*首先，Android显示系统中，系统默认的渲染器是OpenGL，混合使用skia，各个厂商可能有不同的实现，大部分都是OpenGL。   Android在系统启动时，经过BootLoader启动、kernel启动后，进入第三阶段，init进程会启动一系列的核心进程，其中包括ServiceManager、zygote、OpenGL，OpenGL是显示系统的基础，OpenGL启动之后，就可以播放动态的开机动画了。所以说，Android渲染的基础的OpenGL，而**OpenGL渲染管线的最后一个阶段，就是帧缓冲区**（FrameBuffer）。这也是文章讨论的重点。*

# FBO的现实需求：
前面提到，OpenGL渲染管线的最后一个阶段，就是帧缓冲区（FrameBuffer）。对于帧缓冲区，Android系统中默认有一个缓冲区，英文名称window-system-provided framebuffer，用于屏幕显示。  当GPU往显示缓冲区写入数据后，屏幕会显示这个缓冲区的内容。这是一个生产者消费者模型，大多数情况下能满足我们的需求。  
而对于Android某些特殊需求来说，比如给视频加水印后渲染保存，比如Camera实时滤镜。  这些都需要把视频的原始data经过一定的处理后保存、或者再进行显示。
 
对于一般的Camera来说，我们用于预览/录制的Surface都是由Android系统提供给OpenGL渲染的帧缓冲区，OpenGL的所有渲染结果都是直接到达到帧缓冲区，这是on-screen的渲染方式；
而对于滤镜相机、贴纸相机来说，使用帧缓冲区对象，OpenGL可以将原先绘制到窗口提供的帧缓冲区重定向到FBO之中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190614165351247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZyYWtpZV9Ld29r,size_16,color_FFFFFF,t_70)

# FBO的理论：
和窗口提供的帧缓冲区类似，FBO提供了一系列的缓冲区，包括颜色缓冲区、深度缓冲区和模板缓冲区。这些逻辑的缓冲区在FBO中被称为 framebuffer-attachable说明它们是可以绑定到FBO的对象数组上。

FBO中有两类绑定的对象：纹理图像（texture images）和渲染图像（renderbuffer images）。如果纹理对象绑定到FBO，那么OpenGL就会执行渲染到纹理（render to texture）的操作，如果渲染对象绑定到FBO，那么OpenGL会执行离屏渲染(offscreen rendering)
FBO可以理解为包含了许多挂接点的一个对象，它自身并不存储图像相关的数据，他提供了一种可以快速切换外部纹理对象和渲染对象挂接点的方式，在FBO中必然包含一个深度缓冲区挂接点和一个模板缓冲区挂接点，同时还包含许多颜色缓冲区挂节点（具体多少个受OpenGL实现的影响，可以通过GL_MAX_COLOR_ATTACHMENTS使用glGet查询），FBO的这些挂接点用来挂接纹理对象和渲染对象，这两类对象中才真正存储了需要被显示的数据。FBO提供了一种快速有效的方法挂接或者解绑这些外部的对象，对于纹理对象使用 glFramebufferTexture2D，对于渲染对象使用glFramebufferRenderbuffer
FBO。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061210592490.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZyYWtpZV9Ld29r,size_16,color_FFFFFF,t_70)

白话文解释：FBO是一个挂接器，类似画家画画用的托架；其中FBO只能挂接两种对象，纹理图像 和 渲染图像，这个理解就是画家准备创作作品前，在托架上放的是油画纸还是水墨纸（纹理），或者根本不是放画纸，放的是木板雕刻（渲染模板）。最后我们等画家创作出他的艺术品后，直接搬到到展示区，呈现給大家。
