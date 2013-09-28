---
layout:    post
title:     Effective Objective-C Chapter 1
category:  blog
description: 了解 Objective－C
tags: ObjC
---



## 条目 1：熟悉 Objective-C 的根源
Objective-C 和其他的面向对象语言，如 C++ 以及 Java，很相似，但也有很多不同之处。如果你有其他面向对象语言的经验，你将能理解很多 Objective-C 里的范式和模式。然而，它的语法可能看起来有点异类，因为使用了消息分发机制（messaging structure）而不是函数调用。Objective-C 由 Smalltalk 发展而来，而 Smalltalk 则是消息分发的起源。消息分发和函数调用的区表看起来像这样：

	// Messaging (Objective-C)
	Object *obj = [Object new];
	[obj performWith:parameter1 and:parameter2];
	
	// Function calling (C++)
	Object *objc = new Object;
	obj->perform(parameter1, parameter2);
	
重要的不同是，消息分发机制是由运行时决定什么代码被执行的。而函数调用，则由编译器决堤什么代码将会被执行。当多态被引入函数调用的例子中时，便加了一个运行时会被查询的表单，这就是所谓的虚表（virtual table）。但是在消息分发里，查询总是在运行时。实际上，编译器甚至不关心被发送了消息的对象的类型。通过一个动态绑定的过程，对象类型在运行时才决定。动态绑定的更多细节将在条目 11 中描述。

Objective-C 的运行时部分，而不是编译器，承担了大部分繁重的工作。运行时包含了所有的数据结构和使 Objective-C 的面向对象功能工作起来的方法。例如，运行时包括了所有的内存管理方法。实质上，运行时是一个代码集合，它将你的所有代码和你的代码里以动态链接库形式连接的外部代码粘合在一起。从而，无论何时运行时更新了，你的应用都可以从运行时的性能改善中获益。将更多工作放在编译时的语言则需要重新编译才能从这样的运行时性能改善中获益。

Objective-C 是 C 的超集，所以在写 Objective-C 时，C 语言的所有功能都是可用的。因此，为了写出高效的 Objective-C 代码，你需要对 C 和 Objective-C 的核心概念都能理解。特别地，理解 C 的内存模型将帮助你理解 Objective-C 的内存模型和引用计数的工作方式。这涉及到理解在 Objective-C 中指针被用于表示对象。当你声明了一个变量，这个变量持有一个指向一个对象的引用，语句就像这样：

	NSString *someSting = @"The string";

这条语句，大部分直接取自 C，声明了一个名为 someString 的变量，它的类型是 NSString* 。这意味着它是一个指向 NSString 的指针。所有的 Objective-C 对象必须被如此声明，因为对象的内存都分配在堆空间，而从不会在栈中。声明一个分配在栈的 Objective-C 对象是非法的：
	
	NSString stackString;
	// interface type cannot be statically allocated
	
变量 someString 指向了某块分配在堆中的内存，内存中包含了一个 NSString 对象。着意味着创建另外一个指向同一位置的变量将不会创建一块内存拷贝，而只是使两个变量都指向同一对象：

	NSString *someString = @"The string";	NSString *anotherString = someString;	

![alt stack-heap](/images/blog/EffectiveObjC/Stack-Heap-Allocated.png "Stack Heap")

图 1.1 内存分布表示了一个在堆分配的 NSString  实例和两个在栈分配的指向该实例的指针

这里只有一个 NSString	实例，但由两个指向相同实例的变量。这两个变量是 NSString* 类型，意味着当前栈帧分配了2 块指针大小的内存（每块指针的内存大小：32 位构架中是 4 字节，64 位构架中是 8 字节）。这些内存块包含了相同的值：NSString 实例的内存地址。

图 1.1 表达了这种布局。储存 NSString 实例的数据包含了代表实际字符串的字节。

分配在堆中的内存必须被直接管理，而栈中分配的内存中存储的变量会在栈帧被弹出时被自动清理掉。

堆的内存管理被 Objective-C 抽象了出来。你不必使用 malloc 和 free 去分配和销毁对象内存。Objective-C 运行时将这些一个叫引用记数的内存管理机制抽象出来了（见条目 29）。

有时在 Objective-C 中，你会遇到在定义中没有 * 的变量，它们可能永乐栈空间。这些变量不持有 Objective-C 对象。一个例子是 CoreGrphics 库里的 CGRect：

	CGRect frame;	frame.origin.x = 0.0f;	frame.origin.y = 10.0f;	frame.size.width = 100.0f;
	frame.size.height = 150.0f;	
CGRect是一个 C 结构体，定义如下：

	struct CGRect {
		CGPoint origin;
		CGSize size;	};	typedef struct CGRect CGRect;

这些结构体类型被广泛用于系统库中，系统库使用 Objective-C 对象的开销可能会影响性能。而结构体并不会有创建对象产生的开销，例如堆内存的分配和销毁。当非对象类型（int, float, double, char, 等）是唯一被持有的数据时，结构体，例如 CGRect，常常会被用到。

着手写 Objective-C 之前，我鼓励你阅读一些关于 C 语言的资料，熟悉 C 的语法。如果你直接深入 Objective-C，你可能会发现自己对它的一些语法感到困惑。

### 记住

* Objective-C 是 C 的超集，加入了面向对象特性。Objective-C 使用动态绑定的消息分发机制，这意味着对象的类型是在运行时才被发现的。运行时，而不是编译器，决定了那段代码会在接收到消息后运行。

* 理解 C 的核心概念将有助于你写出高效的 Objective-C 代码。特别是要理解内存模型和指针。


















