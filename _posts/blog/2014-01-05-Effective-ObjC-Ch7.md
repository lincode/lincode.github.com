---
layout:    post
title:     Effective Objective-C 2.0 Chapter 7
category:  blog
description: 系统库
tags: ObjC
---

# 第 7 章 系统库
虽然在没有系统库的情况下使用 Objective-C 仍然是可能的，但是这么做是极为少见的。甚至标准根类，NSObject，也是 Foundation 库的一部分，而不是语言的一部分。如果你不用 Foundation 库，你必须写自己的根类，和自己的集合类，事件循环，和其他有用的类。而且，没有系统库，你还不能使用 Objective-C 开发 Mac OS X 和 iOS 应用。它们经过多年才发展成今天这样功能强大。因此，你可能会发现它们某些部分有些古老，用起来有些别扭，但是你也可能会发现瑰宝。


## 条目 47 ： 使你自己熟悉系统库

## 条目 48 ： 枚举块优于循环

## 条目 49 ： 在集合类的自定义内存管理中使用桥接

## 条目 50 ： 缓存中使用 NSCache 替代 NSDictionary

## 条目 51 ： 保持 initialize 和 load 实现的简洁

## 条目 52 ： 记住 NSTimer 会持有它的目标