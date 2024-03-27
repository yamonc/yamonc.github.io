---
title: JVM类加载过程
date: 2023-07-14 15:28:30
tags: [Java, 类加载过程]
categories: [JVM]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307141527134.jpeg
comment: 是
keywords: 
---
# JVM类加载过程

![a small boat floating on top of a large body of water](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307141527134.jpeg)

引入前提：

JVM类加载机制：

类从被加载到JVM中开始，到结束释放内存为止，整个声明周期可以简单概括为7个阶段：

加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）。其中，验证、准备和解析这三个阶段可以统称为连接（Linking）。

这七个阶段的顺序如下图：

![一个类的完整生命周期](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307141027538.png)

