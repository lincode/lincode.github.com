---
layout:    post
title:     Effective Objective-C Chapter 1
category:  blog
description: 了解 Objective-C
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


	NSString *someString = @"The string";
    NSString *anotherString = someString;	

![alt stack-heap](/images/blog/EffectiveObjC/Stack-Heap-Allocated.png "Stack Heap")

图 1.1 内存分布表示了一个在堆分配的 NSString  实例和两个在栈分配的指向该实例的指针

这里只有一个 NSString	实例，但由两个指向相同实例的变量。这两个变量是 NSString* 类型，意味着当前栈帧分配了2 块指针大小的内存（每块指针的内存大小：32 位构架中是 4 字节，64 位构架中是 8 字节）。这些内存块包含了相同的值：NSString 实例的内存地址。

图 1.1 表达了这种布局。储存 NSString 实例的数据包含了代表实际字符串的字节。

分配在堆中的内存必须被直接管理，而栈中分配的内存中存储的变量会在栈帧被弹出时被自动清理掉。

堆的内存管理被 Objective-C 抽象了出来。你不必使用 malloc 和 free 去分配和销毁对象内存。Objective-C 运行时将这些一个叫引用记数的内存管理机制抽象出来了（见条目 29）。

有时在 Objective-C 中，你会遇到在定义中没有 * 的变量，它们可能永乐栈空间。这些变量不持有 Objective-C 对象。一个例子是 CoreGrphics 库里的 CGRect：


	CGRect frame;
    frame.origin.x = 0.0f;
    frame.origin.y = 10.0f;
    frame.size.width = 100.0f;
	frame.size.height = 150.0f;

CGRect是一个 C 结构体，定义如下：

	struct CGRect {
		CGPoint origin;
		CGSize size;
	};
    typedef struct CGRect CGRect;

这些结构体类型被广泛用于系统库中，系统库使用 Objective-C 对象的开销可能会影响性能。而结构体并不会有创建对象产生的开销，例如堆内存的分配和销毁。当非对象类型（int, float, double, char, 等）是唯一被持有的数据时，结构体，例如 CGRect，常常会被用到。

着手写 Objective-C 之前，我鼓励你阅读一些关于 C 语言的资料，熟悉 C 的语法。如果你直接深入 Objective-C，你可能会发现自己对它的一些语法感到困惑。

### 记住

* Objective-C 是 C 的超集，加入了面向对象特性。Objective-C 使用动态绑定的消息分发机制，这意味着对象的类型是在运行时才被发现的。运行时，而不是编译器，决定了那段代码会在接收到消息后运行。

* 理解 C 的核心概念将有助于你写出高效的 Objective-C 代码。特别是要理解内存模型和指针。


## 条目 2：减少在头文件引入头文件

在 Objective-C，就像 C 和 C++ 一样，使用头文件和实现文件。当在 Objective-C 中实现一个类时，标准方法是各创建一个与类同名的头文件和实现文件，头文件以 .h 为后缀名，实现文件以 .m 为后缀名。当你创建一个类时，它完成时应该像这样：

	// EOCPerson.h
	#import <Foundation/Foundation.h>

	@interface EOCPerson : NSObject
	
	@property (nonatomic, copy) NSString *firstName;
	@property (nonatomic, copy) NSString *lastName;
    @end

    // EOCPerson.m
    #import "EOCPerson.h"
    @implementation EOCPerson
    // Implementation of methods
    @end

在 Objective-C 中创建类要求 Foundation.h 的引入是十分普遍的。或者，你会引入一个库的基础头文件，你继承的类就在这个库中。例如，如果你正在创建一个 iOS 应用，你常创建一些继承了 UIViewController 的类。这些类会引入 UIKit.h。

如其代表的，这个类很细微。它却引入了整个 Foundation 库，但着没关系。鉴于这个类继承了一个 Foundation 库中的类，库的大部分功能可能会被 EOCPerson 的使用者用到。同样的事情发生在继承了 UIViewController 的类上。它的使用者都会用到 UIKit。

随着时间的推移，你可能创建了一个名为 EOCEmploer 的新类。然后，你决定 EOCPerson 的实例应该拥有另一个 EOCPerson 的实例。所以，你继续为类添加一个属性：

      // EOCPerson.h	
      #import <Foundation/Foundation.h>
	  @interface EOCPerson : NSObject
      @property (nonatomic, copy) NSString *firstName;
      @property (nonatomic, copy) NSString *lastName;
      @property (nonatomic, strong) EOCEmployer *employer;
      @end

这其中有一个问题，在编译任何包含了 EOCPerson 的文件时，类 EOCEmlpoyer 并不是可见的。要求所有人在引入 EOCPerson.h 的时候也引入 EOCEmployer.h 是不对的。所以通常的做法是在 EOCPerson.h 顶部加入如下语句：

      #import "EOCEmployer.h"

这可行，但却是糟糕的做法。为了编译任何用到了 EOCPerson 的东西，你无需知道 EOCEmployer 的所有实现细节。所有你只需要知道有一个名为 EOCEmployer 的类存在即可。幸运的是，确实存在告诉编译器这么做的方法：

	@class EOCEmployer;

这叫做前向声明。EOCPerson 的头文件的结果将会像这样：

	// EOCPerson.h	
	#import <Foundation/Foundation.h>	
	@class EOCEmployer;
	
	@interface EOCPerson : NSObject
	@property (nonatomic, copy) NSString *firstName；
    @property (nonatomic, copy) NSString *lastName; 
	@property (nonatomic, strong) EOCEmployer *employer;
	@end

EOCPerson 的实现文件之后将需要引入 EOCEmployer 的头文件，因为它需要知道类的全部接口细节以使用这个类。所以实现文件完成时像这样：


	// EOCPerson.m	
	#import "EOCPerson.h"	
	#import "EOCEmployer.h"
	@implementation EOCPerson	
	// Implementation of methods 	
	@end


将引入推迟到它正真需要的地方，可以使你限制你的类的使用需要引入的类的范围。如例子中，如果 EOCEmployer.h 被 EOCPerson.h 引入了，所有引入了 EOCPerson.h 的文件都会引入整个 EOCEmployer.h 文件。如果引入链继续，你可能引入大大多于你需要的文件，这将显著增加编译时间。

使用前向声明同时可以缓解两个类相互引用的问题。考虑一下如果 EOCEmployer 有添加和删除雇员的方法将会发生什么，其在头文件中定义如下：


	- (void)addEmployee:(EOCPerson*)person;	
	- (void)removeEmployee:(EOCPerson*)person;

这时，类 EOCPerson 必须对编译器可见，在相对的情况也是如此 (编译 EOCPerson时，EOCEmployer 必须对编译器可见)。可是，通过在头文件引入一个需要引入自己的头文件的方法会产生鸡和蛋问题。但一个头文件被解析时，它引入了另一个头文件，另一个头文件却引入了前一个头文件。使用 #import 而非 #include 将无法中止这个无限循环，没有一个类可以被正确编译。自己试一下，不要完全相信我！

有时，你仍然需要在一个头文件中引入另外一个头文件。你必须引入定义了你所继承的那个类的头文件。类似地，如果你声明了你的类遵守某个协议，那么你将需要协议的完整定义而不是前向声明。便已去需要可以看到协议定义的方法而不是简单地从前向声明中知道协议的存在。

例如，假定一个类 rectangle 继承了类 shape，并遵守一个允许其画图的协议：

	// EOCRectangle.h
	#import "EOCShape.h"
	#import "EOCDrawable.h"
	
	@interface EOCRectangle : EOCShape <EOCDrawable>
	@property (nonatomic, assign) float width;
	@property (nonatomic, assign) float height;
	@end

额外的引入是不可避免的。因为协议而引入包含协议的头文件时要慎重。如果协议 EOCDrawable 是一个很大的头文件的一部分，你必须将它门全部引入，从而制造了相同的依赖和前面所描述的额外的编译时间。
不是所有的协议，例如委托协议（见条目 23），都需要一个自己单独的文件。在这样例子中，作为类的委托协议，只有和被它委托的类比肩在一起，协议才是有意义的。在这种情况下，最好声明你会在类的目录中实现委托（见条目 27）。这意味着可以在实现文件中，而不是在公开的头文件中，引入包含委托协议的头文件。 
当在头文件中写入一个引入时，总要问一下自己是否是真的需要的。如果引入可以被前向声明代替，那么就前向声明。如果为了属性，实例变量，或者遵守某个协议而引入头文件，那么可以将其移到类的目录（见条目 27）中。这么做，可以保证编译时间尽可能的短，并减少起维护问题的相互依赖，并减少在公共 API 中过多暴露出你的代码的问题。这些都是你应该要做到的。

### 记住

* 总是尽可能拖后引用头文件。这通常意味着在头文件中使用前向声明，而在实现文件中引入对应的头文件。这样做可以做到尽可能地避免类之间的耦合。

* 有时，无法做到前向声明，比如当声明遵守协议的时候。这种情况下，如果可能，考虑把遵守协议的声明放入类的目录中。否则，只引入定义了协议的头文件。


## 条目 3：文字语法优于与其等价的方法
在使用 Objective-C 时，你时常会遇到一些类。它们都是 Foundation 库的一部分。虽然技术上来讲，你并不是必须使用 Foundation 库来写 Objective-C 代码，但是实践中，你通常会用上它。这些类是 NSString, NSNumber, NSArray, 和 NSDictionary。数据结构从名字上看便知道其用途。

Objective-C 以复杂的语法著称。这时真的。但是，自从 Objective-C 1.0 开始，就有非常便捷的方法创建一个 NSString 对象。这就是文字字符串，看起来如下：

	NSString *someString = @"Effective Objective-C 2.0";

如果没有这个语法，创建一个 NSString 对象将需要以惯常的 alloc 和 init 方法调用来分配和初始化一个 NSString 对象。幸运的是，这个语法，以文字描述，在近几个版本的编译器中被扩展到了 NSNumber, NSArray, NSDictionary。使用文字语法可以减少代码量，使得代码更易读。

### 文字语法数字

有时，你需要将整数，浮点数，布尔值包装为一个 Objective-C 对象。你可以使用 NSNumber 类来完成这项工作，它能处理一系列数字类型。如果没有文字语法，你需要如此创建一个数字对象：

	NSNumber *someNumber = [NSNumber numberWithInt:1];

这创建了一个整数，其值设为 1。但，使用文字语法将使它更清楚：

	NSNumber *someNumber = @1;
	
如你所见，文字语法更为简洁。但是，不仅仅如此。语法同时也覆盖所有 NSNumber 实例可以表达的数据类型。例如：

	NSNumber *intNumber = @1;
	NSNumber *floatNumber = @2.5f;
    NSNumber *doubleNumber = @3.14159;
    NSNumber *boolNumber = @YES;
    NSNumber *charNumber = @'a';

文字语法也可以如此表达：


	int x = 5;
    float y = 6.32f;
    NSNumber *expressionNumber = @(x * y);

使用数字的文字语法是非常有用的。这么做使得 NSNumber 对象更为清楚，因为，声明的大部分本身就是数值，而不是繁复的语法。

### 文字语法数组
数组是一个常用数据结构。没有文字语法，你需要如此创建一个数组：

	NSArray *animals = [NSArray arrayWithObjects:@"cat", @"dog", 
												  @"mouse", @"badger", nil];

使用文字语法，只要求如下语句：

	NSArray *animals = @[@"cat", @"dog", @"mouse", @"badger"];

虽然这已是个相当简单的语法，但仍不仅如此。一个普遍的操作是取数组特定下标下的对象。使用文字语法也能简化这个操作。通常的，你会使用 objectiveAtIndex: 方法：

	NSString *dog = [animals objectAtIndex:1];

使用文字语法，就变成如下代码：

	NSString *dog = animals[1];

下标，如同其他文字语法一样使得要做的事情看起来更为简洁。并且，它看起来也和其他语言的数组索引语法类似。

可以，使用文字语法创建数组时，你需要注意一件事情。如果任何对象是 nil，一个异常将会被抛出。因为文字语法只是一个语法糖，它创建数组并向数组添加方括号里的所有对象。你看到的异常会是这样：

	*** Terminating app due to uncaught exception	'NSInvalidArgumentException', reason: '***	-[__NSPlaceholderArray initWithObjects:count:]: attempt to	insert nil object from objects[0]'

这表明了一个使用文字语法的共同问题。下面的代码创建了两个数组，使用了两种语法：
        
   id object1 = /* ... */;
   id object2 = /* ... */;
   id object3 = /* ... */;

   NSArray *arrayA = [NSArray arrayWithObjects:object1, object2, object3, nil];	NSArray *arrayB = @[object1, object2, object3];

想在考虑一下这样一个场景：object1 和 object3 指向一个合法 Objective-C 对象，但是 object2 是 nil。文字语法数组，arrayB，将导致异常被抛出。可是，arrayA 仍然会被创建，但只包含 object1。原因是方法 arrayWithObjects: 浏览可变参数直到遇到 nil。这个场景中，比预期的早了一些。

这个微妙的不同意味着文字语法更为安全。抛出异常可能导致应用崩溃，比创建一个比预期中包含了对象更少的数组，要更好一些。一个程序员的错误最有可能导致向数组插入 nil，异常的抛出使得 bug 更容易被发现。

### 文字语法字典
字典提供了一个在图中添加键值对的数据结构。如数组一样，字典在 Objective-C 代码中使用得很广泛。过去如此创建字典：

	NSDictionary *personData = [NSDictionary dictionaryWithObjectsAndKeys:@"Matt", @"firstName", 
                               @"Galloway", @"lastName", 
                               [NSNumber numberWithInt:28], @"age", nil];

这相当令人困惑，因为顺序是 <object>，<key>，<object>，<key>，等等。可是，你常常会以键到值得方式思考一个字典。因此，它相当不好阅读。可是，文字语法再次提供了更清晰的语法：

	NSDictionary *personData = @{@"firstName" : @"Matt",
                                 @"lastName" : @"Galloway",
                                 @"age" : @28};

这非常简洁，键总是在值之前，如果你期望的那样。同时，注意到数字也可以文字语法的形式表达，这相当有用。对象和值都必须是 Objective-C 对象，所以你不能存储整数 28；必须要讲起包装为一个 NSNumber 实例。但是文字语法意味这简单的只是多了一个额外的字符。

就像数组一样，字典的文字语法在插入任何 nil 值时，也会抛出异常。可是，基于相同的理由，这是件好事。这意味，如果通过遇到 nil 会停止浏览的 dictionaryWithObjectsAndKeys: 创建字典会丢失一些值，而文字语法会抛出异常。

同时和数组类似，字典可以通过文字语法访问。旧的通过特定键值访问的方法如下：

	NSString *lastName = [personData objectForKey:@"lastName"];

对应的文字语法：
	
	NSString *lastName = personData[@"lastName"];

再一次，多余的语句减少了，留下了易读的代码。


### 可变数组和字典
用相同的方法，你可以通过下标索引数组和访问字典。如果它是可变的，你也可以为其赋值。通过普通方法为数组和字典赋值，就像这样：

	[mutableArray replaceObjectAtIndex:1 withObject:@"dog"];
    [mutableDictionary setObject:@"Galloway" forKey:@"lastName"];

通过下标赋值，则像这样：

	mutableArray[1] = @"dog";
    mutableDictionary[@"lastName"] = @"Galloway";

### 限制
文字语法的一个小限制是，除了字符串，创建的对象必须来自 Foundation 库。没有办法指定你自己的子类通过文字语法来创建。如果你要创建一个自己的子类的实例，你需要使用非文字语法。可是，因为 NSArray，NSDictionary，和 NSNumber 是类簇（见条目 9），它们很少被继承，因为，这么做并不简单。同时，标准实现也已经足够好了。String 可以使用客户类，但是必须改变编译器选项。使用这个选择是不被鼓励的，除非你知道你在做什么。

同时，在字符串，数组，字典的情况下，只有不可变变量可以被用于文字语法的创建过程。如果要求可变变量，则制作一个可变变量的拷贝，就像这样：

	NSMutableArray *mutable = [@[@1, @2, @3, @4, @5] mutableCopy];

这里添加了一个额外的方法调用，一个额外的对象被创建了。因此，虽然使用文字语法有好处，但在这儿有些得不偿失。

### 记住

* 使用文字语法创建字符串，数字，数组，和字典。这比使用普通的创建对象方法更为清楚和简洁。

* 索引数组或者在字典中取值使用下标方法。

* 试图向数组或者字典插入 nil 会导致异常被抛出。因此，要确认插入值不为 nil。  

## 条目 4：常量优于预处理 ＃define


## 条目 5：表达状态，可选项，状态码时使用枚举






