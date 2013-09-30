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
		CGSize size;	};	typedef struct CGRect CGRect;

这些结构体类型被广泛用于系统库中，系统库使用 Objective-C 对象的开销可能会影响性能。而结构体并不会有创建对象产生的开销，例如堆内存的分配和销毁。当非对象类型（int, float, double, char, 等）是唯一被持有的数据时，结构体，例如 CGRect，常常会被用到。

着手写 Objective-C 之前，我鼓励你阅读一些关于 C 语言的资料，熟悉 C 的语法。如果你直接深入 Objective-C，你可能会发现自己对它的一些语法感到困惑。

### 记住

* Objective-C 是 C 的超集，加入了面向对象特性。Objective-C 使用动态绑定的消息分发机制，这意味着对象的类型是在运行时才被发现的。运行时，而不是编译器，决定了那段代码会在接收到消息后运行。

* 理解 C 的核心概念将有助于你写出高效的 Objective-C 代码。特别是要理解内存模型和指针。


## 条目 1：减少在头文件引入头文件

在 Objective-C，就像 C 和 C++ 一样，使用头文件和实现文件。当在 Objective-C 中实现一个类时，标准方法是各创建一个与类同名的头文件和实现文件，头文件以 .h 为后缀名，实现文件以 .m 为后缀名。当你创建一个类时，它完成时应该像这样：

	// EOCPerson.h
	#import <Foundation/Foundation.h>
	
	@interface EOCPerson : NSObject	@property (nonatomic, copy) NSString *firstName;	@property (nonatomic, copy) NSString *lastName;	@end
	// EOCPerson.m	#import "EOCPerson.h"
	@implementation EOCPerson	// Implementation of methods 	@end
	在 Objective-C 中创建类要求 Foundation.h 的引入是十分普遍的。或者，你会引入一个库的基础头文件，你继承的类就在这个库中。例如，如果你正在创建一个 iOS 应用，你常创建一些继承了 UIViewController 的类。这些类会引入 UIKit.h。
如其代表的，这个类很细微。它却引入了整个 Foundation 库，但着没关系。鉴于这个类继承了一个 Foundation 库中的类，库的大部分功能可能会被 EOCPerson 的使用者用到。同样的事情发生在继承了 UIViewController 的类上。它的使用者都会用到 UIKit。随着时间的推移，你可能创建了一个名为 EOCEmploer 的新类。然后，你决定 EOCPerson 的实例应该拥有另一个 EOCPerson 的实例。所以，你继续为类添加一个属性：
	// EOCPerson.h	#import <Foundation/Foundation.h>	@interface EOCPerson : NSObject	@property (nonatomic, copy) NSString *firstName;	@property (nonatomic, copy) NSString *lastName;	@property (nonatomic, strong) EOCEmployer *employer;	@end
这其中有一个问题，在编译任何包含了 EOCPerson 的文件时，类 EOCEmlpoyer 并不是可见的。要求所有人在引入 EOCPerson.h 的时候也引入 EOCEmployer.h 是不对的。所以通常的做法是在 EOCPerson.h 顶部加入如下语句：
	#import "EOCEmployer.h"
这可行，但却是糟糕的做法。为了编译任何用到了 EOCPerson 的东西，你无需知道 EOCEmployer 的所有实现细节。所有你只需要知道有一个名为 EOCEmployer 的类存在即可。幸运的是，确实存在告诉编译器这么做的方法：

	@class EOCEmployer;这叫做前向声明。EOCPerson 的头文件的结果将会像这样：
	// EOCPerson.h	#import <Foundation/Foundation.h>		@class EOCEmployer;
	@interface EOCPerson : NSObject	@property (nonatomic, copy) NSString *firstName;	@property (nonatomic, copy) NSString *lastName; 	@property (nonatomic, strong) EOCEmployer *employer; 	@end	
EOCPerson 的实现文件之后将需要引入 EOCEmployer 的头文件，因为它需要知道类的全部接口细节以使用这个类。所以实现文件完成时像这样：

	// EOCPerson.m	#import "EOCPerson.h"	#import "EOCEmployer.h"
	@implementation EOCPerson	// Implementation of methods 	@end

将引入推迟到它正真需要的地方，可以使你限制你的类的使用需要引入的类的范围。如例子中，如果 EOCEmployer.h 被 EOCPerson.h 引入了，所有引入了 EOCPerson.h 的文件都会引入整个 EOCEmployer.h 文件。如果引入链继续，你可能引入大大多于你需要的文件，这将显著增加编译时间。

使用前向声明同时可以缓解两个类相互引用的问题。考虑一下如果 EOCEmployer 有添加和删除雇员的方法将会发生什么，其在头文件中定义如下：

	- (void)addEmployee:(EOCPerson*)person;	- (void)removeEmployee:(EOCPerson*)person;

这时，类 EOCPerson 必须对编译器可见，在相对的情况也是如此 (编译 EOCPerson时，EOCEmployer 必须对编译器可见)。可是，通过在头文件引入一个需要引入自己的头文件的方法会产生鸡和蛋问题。但一个头文件被解析时，它引入了另一个头文件，另一个头文件却引入了前一个头文件。使用 #import 而非 #include 将无法中止这个无限循环，没有一个类可以被正确编译。自己试一下，不要完全相信我！

有时，你仍然需要在一个头文件中引入另外一个头文件。你必须引入定义了你所继承的那个类的头文件。类似地，如果你声明了你的类遵守某个协议，那么你将需要协议的完整定义而不是前向声明。便已去需要可以看到协议定义的方法而不是简单地从前向声明中知道协议的存在。

例如，假定一个类 rectangle 继承了类 shape，并遵守一个允许其画图的协议：

	// EOCRectangle.h	#import "EOCShape.h"	#import "EOCDrawable.h"
	@interface EOCRectangle : EOCShape <EOCDrawable> 	@property (nonatomic, assign) float width; 	@property (nonatomic, assign) float height;	@end额外的引入是不可避免的。因为协议而引入包含协议的头文件时要慎重。如果协议 EOCDrawable 是一个很大的头文件的一部分，你必须将它门全部引入，从而制造了相同的依赖和前面所描述的额外的编译时间。
不是所有的协议，例如委托协议（见条目 23），都需要一个自己单独的文件。在这样例子中，作为类的委托协议，只有和被它委托的类比肩在一起，协议才是有意义的。在这种情况下，最好声明你会在类的目录中实现委托（见条目 27）。这意味着可以在实现文件中，而不是在公开的头文件中，引入包含委托协议的头文件。 
当在头文件中写入一个引入时，总要问一下自己是否是真的需要的。如果引入可以被前向声明代替，那么就前向声明。如果为了属性，实例变量，或者遵守某个协议而引入头文件，那么可以将其移到类的目录（见条目 27）中。这么做，可以保证编译时间尽可能的短，并减少起维护问题的相互依赖，并减少在公共 API 中过多暴露出你的代码的问题。这些都是你应该要做到的。
### 记住

* 总是尽可能拖后引用头文件。这通常意味着在头文件中使用前向声明，而在实现文件中引入对应的头文件。这样做可以尽量避免类之间的耦合。

* 有时，无法做到前向声明，比如当声明遵守协议的时候。这种情况下，如果可能，考虑把遵守协议的声明放入类的目录中。否则，只引入定义了协议的头文件。













