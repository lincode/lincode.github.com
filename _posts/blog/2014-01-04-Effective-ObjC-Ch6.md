---
layout:    post
title:     Effective Objective-C 2.0 Chapter 6
category:  blog
description: Block 和 Grand Central Dispatch
tags: ObjC
---

# 第 6 章 Block 和 Grand Central Dispatch
多线程编程是每个开发者在进行现代应用开发时需要考虑的。即使你不认为你的应用是多线程的，这更可能是因为系统库经常使用 UI 线程之外的额外线程完成工作。没有什么比 UI 线程被卡住更糟糕了。在 Mac OS X 中，这会导致可怕的菊花一直转；在 iOS 中，你的应用可能会被终结，如果它卡死太长时间的话。

幸运的是，Apple 已用一个全新的方法实现了多线程。现代多线程的核心功能是 block 和 Gradn Central Dispatch (GCD)。虽然是不同的分立的技术，但是会一起介绍它们。Blocks 给 C，C++，Objective-C 提供了闭包。闭包很有用，这主要是因为它们提供了一种将代码如同对象一样传递，使得它们能在不同的上下文运行的机制。更关键的是，Blocks 可以使用任何它所被定义的作用域中的任何变量。

GCD 是一系列相关技术，基于所谓的分发队列，提供了对线程的抽象。闭包可以被排入这些队列，GCD 为你排期运行。GCD 创建，重用，销毁后台线程，基于系统资源以其觉得合适的方式运行每个队列。而且，GCD 提供易于使用的解决方案给通常的编程任务，例如线程安全单一运行代码，基于可能的系统资源并行执行任务。

Blocks 和 GCD 都是现代 Objective-C 编程的支柱。因此，你需要理解它们如何工作以及它们提供了什么功能。


## 条目 37 ： 理解 Blocks

## 条目 38 ： 为公共 Block 通过 typedefs 创建类型

## 条目 39 ： 用 Blocks 回调减少代码分离

## 条目 40 ： 避免 block 内强引用的对象引入的 Retain 循环

## 条目 41 ： 推荐使用 dispatch queues 来同步锁

## 条目 42 ： 推荐 GCD 而不是 performSelector 系列方法

## 条目 43 ： 知道何时使用 GCD 和 何时使用操作队列

## 条目 44 ： 使用 dispatch group 以利用平台可扩张性

## 条目 45 ： 使用 dispatch_once 执行有线程安全要求的代码

## 条目 46 ： 避免 dispatch_get_current_queue

 