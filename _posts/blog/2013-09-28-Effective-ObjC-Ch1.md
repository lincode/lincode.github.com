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

### 要点回顾

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

### 要点回顾

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

	*** Terminating app due to uncaught exception 'NSInvalidArgumentException', 
    reason: '***-[__NSPlaceholderArray initWithObjects:count:]: attempt to insert nil object from objects[0]'

这表明了一个使用文字语法的共同问题。下面的代码创建了两个数组，使用了两种语法：

   id object1 = /* ... */;
   id object2 = /* ... */;
   id object3 = /* ... */;

   NSArray *arrayA = [NSArray arrayWithObjects:object1, object2, object3, nil];
   NSArray *arrayB = @[object1, object2, object3];

想在考虑一下这样一个场景：object1 和 object3 指向一个合法 Objective-C 对象，但是 object2 是 nil。文字语法数组，arrayB，将导致异常被抛出。可是，arrayA 仍然会被创建，但只包含 object1。原因是方法 arrayWithObjects: 浏览可变参数直到遇到 nil。这个场景中，比预期的早了一些。

这个微妙的不同意味着文字语法更为安全。抛出异常可能导致应用崩溃，比创建一个比预期中包含了对象更少的数组，要更好一些。一个程序员的错误最有可能导致向数组插入 nil，异常的抛出使得 bug 更容易被发现。

### 文字语法字典
字典提供了一个在图中添加键值对的数据结构。如数组一样，字典在 Objective-C 代码中使用得很广泛。过去如此创建字典：

	NSDictionary *personData = 
		[NSDictionary dictionaryWithObjectsAndKeys:@"Matt", @"firstName", 
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

### 要点回顾

* 使用文字语法创建字符串，数字，数组，和字典。这比使用普通的创建对象方法更为清楚和简洁。

* 索引数组或者在字典中取值使用下标方法。

* 试图向数组或者字典插入 nil 会导致异常被抛出。因此，要确认插入值不为 nil。  

## 条目 4：常量优于预处理 ＃define
写代码时，你经常定义一些常量。例如，考虑一个可以通过动画自我呈现和消失的 UI 视图类。一个你很可能会典型常量是动画持续时间。你已学完了关于 Objective-C 和 C 的基础，所以你使用如下方法定义常量：

	#define ANIMATION_DURATION 0.3

这是一个预处理命令；源代码中的任何字符串 ANIMATION_DURATION 都会被替换为 0.3。这看起来似乎正是你需要的，但是这个定义没有类型信息。它似乎是某个被声明为与时间相关的值，但是这并不是显式的。同时，预处理将会盲目地替换为所有它遇到的 ANIMATION_DURATION，所以，如果在头文件中声明了它，引入的其他的头文件也会被替换。

为了解决这个问题，你应该借助编译器。总是有一些比比使用预处理更好的定义常量的方法。例如，一下定义了一个了 NSTimeInterval 类型的常量：

	static const NSTimeInterval kAnimationDuration = 0.3;

注意，使用这种风格，会带上类型信息。这是很有益的，因为它清楚定义了常量是什么。类型是 NSTimeInterval，这也对文档化变量的使用有帮助。如果你有很多变量要定义，这将确实帮助到你和其他之后会阅读到代码的人。

同时，注意到变量是如何命名的。通常的约定是以字母 k 为在作用域（实现文件）里的常量的前缀。对于那些暴露给外部的常量，通常以类名为前缀。条目 19 更详细地解释了命名规则。

在什么地方定义你的常量是十分重要的。有时，你会试图在头文件中声明预处理定义，但这显然是不好的实践，特别是定义名字时还以不会相互避开的方式。例如，ANIMATION_DURATION 常量如果出现在头文件中，可能就是个不好的名字。它可能出现在其它的被引入的头文件中。甚至，以 static const 开头定义的变量也不应该出现在头文件中。因为，Objective-C 没有命名空间，它将声明一个叫 kAnimationDuration 全局变量。它的名字应该以某些字符为前缀，前缀应是用到它的类的类名，例如，EOCViewClassAnimationDuration。条目 19 更详细地解释了如何使清晰地命名。

一个无需向外暴露的常量应该被定义在使用它的实现文件中。例如，如果代表动画持续时间的常量被使用在一个 UIView 的子类中，因为是一个 iOS 应用，所以使用了 UIlit，它看起来就像这样：

	// EOCAnimatedView.h
	#import <UIKit/UIKit.h>

	@interface EOCAnimatedView : UIView 
    - (void)animate;	
    @end	
	
    // EOCAnimatedView.m	
	#import "EOCAnimatedView.h"	

    static const NSTimeInterval kAnimationDuration = 0.3;

	@implementation EOCAnimatedView
    - (void)animate {
        [UIView animateWithDuration:kAnimationDuration
        	             animations:^(){	
    }	
    @end

将变量声明为 static 和 const 是很重要的。const 标示符意味着编译器会抛出一个错误，如果你试图改变常量的值的话。在这个场景中，这正是我们所期望的。常量的值不允许被改变。static 标示符意味着变量是属于其被定义的翻译单元的。一个翻译单元就是一个编译器接收的输入并产生的一个对象文件。在 Objective-C 的例子里，这通常意味着一个类对应一个翻译单元：每个实现文件（.m）。所以在前面的例子中，kAnimationDuration 将被声明为属于 EOCAnicatedView.m 所产生的文件对象。如果变量没有被声明为 static，那么编译器将为其创建一个外部标示。如果其它翻译单元也声明了一个同名变量，链接器会抛出一个和这类似的错误信息：

    duplicate symbol _kAnimationDuration in:
    EOCAnimatedView.o
    EOCOtherView.o

实际上，当声明一个变量为 static 和 const 时，编译器并没有创建一个标示，只是和预处理一样替换定义的变量。但是，记住，好处是有了类型信息。

有时，你会向外暴露常量。例如，如果你的类将要通知其它使用了 NSNotificationCenter 的类，你可能就会这么做。这项工作是由一个对象发布提醒，其它注册过的类则接收提醒。提醒有一个字符串名，这就是你需要显式声明的外部可见的常量。这么做意味这任何要注册接收这个提醒的人都不需要知道提醒的真正地字符串名，而仅仅简单地使用常量即可。

这个常量需要出现在全局标示列表中，以备被其所定义地翻译单元的外部使用。因此，这些常量需要以不同与 static const 的例子的方式声明。这些常量需要如此定义：

	// In the header file
    extern NSString *const EOCStringConstant; 

    // In the implementation file
    NSString *const EOCStringConstant = @"VALUE";

常量在头文件中声明，并在实现文件中定义。在常量的类型中，放置 const 标示符是很重要的。这些定义是向后读的，意味着，在这个例子中，EOCStringConstant 是一个指向 NSString 的指针常量。这正是我们需要的；常量不应该被改变指向其它不同的 NSString 对象。

在头文件中的关键字 extern 告知编译器在引入一个文件时遇到常量时需要做什么。关键字告诉编译器将有一个名为 EOCStringConstant 的标示在全局标示列表中。这意味着这个常量即使在编译器看不到它的定义时，也可被使用。编译器只是简单地知道在二进制文件被链接时，常量会存在。

常量必须且只能被定义一次。通常在和声明它的头文件对应的实现文件中定义它。编译器将会在这个实现文件所生成的文件对象的数据段为这个字符串分配存储空间。当这个文件对象与其它文件对象链接产生二进制文件时，无论其被在哪里使用，链接器将能够解析全局标示 EOCStringConstant。

标示出现在全局标示列表中意味着你为常量命名时应该小心。例如，一个为某个应用处理登录的类可能会在登录完成后发出一个提醒。这个提醒可能看起来像这样：

	// EOCLoginManager.h
    #import <Foundation/Foundation.h>

	extern NSString *const EOCLoginManagerDidLoginNotification;


	@interface EOCLoginManager : NSObject 
	- (void)login;
    @end


	// EOCLoginManager.h
    #import "EOCLoginManager.h"

	NSString *const EOCLoginManagerDidLoginNotification = @"EOCLoginManagerDidLoginNotification";


	@implementation EOCLoginManager

    - (void)login {
    // Perform login asynchronously, then call 'p_didLogin'.
    }

    - (void)p_didLogin {    	
     [[NSNotificationCenter defaultCenter] postNotificationName:EOCLoginManagerDidLoginNotification 
                                                         object:nil];
     }

     @end

注意给常量的名字。与常量相关的类的名字为前缀是个慎重的命名法，这会帮助你避免潜在的冲突。在系统库中，通常如此这样命名。例如，UIkit，以相同的方法将提醒的名字声明为全局变量。这些名字包括 UIApplicationDidEnterBackgroundNotification 和 UIApplicationWillEnterForegroundNotification。

同样的方法也可以用于其他类型的常量。如果前面的例子中的动画持续时间需要暴露给 EOCAnimationView 以外的类，你可以如此声明：

	// EOCAnimatedView.h
    extern const NSTimeInterval EOCAnimatedViewAnimationDuration; 
    
    // EOCAnimatedView.m
    const NSTimeInterval EOCAnimatedViewAnimationDuration = 0.3;

这种方法定义常量比预处理更好，因为编译器可以帮忙确定常量值不会被改变。一旦，定义在 EOCAnimatedView.m，它的值就可以到处使用。一个预处理定义可以因为错误而被重新定义，这意味着不同部分的应用使用了不同的值。

总的来说，避免使用预处理定义常量。作为代替，使用对编译器可见地常量，例如 static const 和实现文件中的全局定义。

### 要点回顾

* 避免预处理定义。它们不包含个人和类型信息，编译器只是简单地查找和替换。他们可能被重定义而无任何警告，在一个应用中产生不同地不一致的值。

* 在实现文件中，以 static const 定义单元明确可翻译的常量。这些常量将不会被暴露在全局符号表中，所以它们的名字无需注意名字空间问题。

* 在头文件中将全局常量定义为外部的，再在相关的实现文件中定义它们。这些常量将出现在全局符号表中。所以，它们的名字应该注意名字空间问题，通常是在它们将它们所属类的名字为前缀。


## 条目 5：表达状态，可选项，状态码时使用枚举

因为 Objective-C 基于 C 语言，所以 C 的所有功能对于 Objective-C 而言都是可用的。其中就有枚举类型，enum。它广泛用于系统库但却常常被开发者忽视。这是一个相当有用的可用于定义常量的方法，例如，一个错误代码可被定义为一组相关的选项。由于 C++ 11 标准，最近版本的系统库包含了定义枚举类型的方法。而 Objective-C 也获益于 C++ 11 标准。

一个枚举仅仅只是一种常量命名方法。一个简单的枚举集合可被用于定义一个对象所经历的状态。例如，socket 连接可以使用如下枚举：

	enum EOCConnectionState { 
	     EOCConnectionStateDisconnected, 
	     EOCConnectionStateConnecting, 
	     EOCConnectionStateConnected,	
    };

使用枚举意味着代码是可读的，因为每个状态都可被隐射为一个易读的值。编译器给每个枚举成员唯一值，从 0 开始每个成员都增加 1。支持这样的枚举的类型是编译器独立的，但是必须至少有足够的位来表达全部枚举值。在前面的例子中，只需要一个 char （一个字节），因为最大值是 2。

这样定义枚举的风格并不是特别有用，因为，它要求如下语法：

    enum EOCConnectionState state = EOCConnectionStateDisconnected;

有一种更为简单的方法，不用每次都输入 enum 而仅仅只要输入枚举名 EOCConnectionState。为了达到这个目的，你需要在定义枚举时增加 typedef：

	enum EOCConnectionState { 
	     EOCConnectionStateDisconnected, 
	     EOCConnectionStateConnecting, 
	     EOCConnectionStateConnected,	
    };
	typedef enum EOCConnectionState EOCConnectionState;

这意味着 EOCConnectionState 可以替代 enum EOCConnectionState 被使用：

	EOCConnectionState state = EOCConnectionStateDisconnected;

C++ 11 标准的到来为枚举类型带来了一些改变。一个变化就是可以指定枚举类型存储的变量的相关类型。这么做的好处是你可以前向声明枚举类型了。不指定相关类型，一个枚举类型不能做前向声明，因为编译器不知道相关类型的大小。因此，当类型被使用时，编译器不能知道应该为变量分配多少空间。

指定类型，你可以使用如下语法：

	enum EOCConnectionStateConnectionState : NSInteger { /* ... */ };

这意味着枚举成员被保证一定是 NSInteger。如果你希望如此，类型可以被如此前向声明：

	enum EOCConnectionStateConnectionState : NSInteger;

也可能定义枚举中特定成员的值，而不是让编译器为你选一个值。语法如下：

	enum EOCConnectionStateConnectionState { 
		EOCConnectionStateDisconnected = 1, 
		EOCConnectionStateConnecting, 
		EOCConnectionStateConnected,	
    };

这意味着 EOCConnectionStateDisconnected 将为 1 而不是 0。后面的值将如前面一样，每次自增 1。因此，EOCConnectionStateConnected 将为 3。

另外一个使用枚举类型的理由是定义选项，特别是选项是相互关联的一组变量时。如果枚举被正确定义，选项可以通过位或操作联合使用。例如，考虑如下一个在 iOS UI 库找到的枚举类型，它用于定义一个视图可以以何种方式被重置大小：

	enum UIViewAutoresizing {	
         UIViewAutoresizingNone = 0,
	     UIViewAutoresizingFlexibleLeftMargin = 1 << 0,
	     UIViewAutoresizingFlexibleWidth = 1 << 1,
	     UIViewAutoresizingFlexibleRightMargin = 1 << 2,
         UIViewAutoresizingFlexibleTopMargin = 1 << 3,	
         UIViewAutoresizingFlexibleHeight = 1 << 4,	
         UIViewAutoresizingFlexibleBottomMargin = 1 << 5,	
    }

使用前述语法，每个选项都可以打开或关闭，因为，每个选项都仅仅使用一位表示。多个选项可以通过位或一同使用：例如，UIViewAutoResizingFlexibleWidth | UIViewAutoresizingFlexibleHeight 。图 1.2 展示了枚举的每个成员的位示意图，和两个联合选项的位示意图。

也可以使用位与操作来确定某个选项是否被设置了：

    enum UIVewAutoresizing resizing = 
		UIViewAutoresizingFlexibleWidth | 	
		UIViewAutoresizingFlexibleHeight;
    if (resizing & UIViewAutoresizingFlexibleWidth) {	 
        // UIViewAutoresizingFlexibleWidth is set	
    }

![alt enum](/images/blog/EffectiveObjC/Enum.png "Enum bit layout")

这广泛用于系统库中。另外一个例子来自于 iOS UI 库 UIKit，使用这个枚举告知系统你的视图所支持的设备朝向。这是通过一个叫 UIInterfaceOrientationMask 枚举类型完成的，你实现一个名为 supportedInterfaceOrientations 的方法，指明所支持的设备朝向：

	- (NSUInteger)supportedInterfaceOrientations {
		return UIInterfaceOrientationMaskPortrait | UIInterfaceOrientationMaskLandscapeLeft;
    }

Foundation 库中两个便利方法，可帮助你定义指定在其中使用整型存储枚举成员的枚举类型。便利方法提供后向兼容，如果你针对新标准的编译器开发的话，新语法被使用；如果不是的话，就退回到旧语法。便利方法以预定义 #define 宏的形式提供。一个例子，定义了如 EOCConnectionState 普通枚举类型。另外一个定义如 UIViewAutoresizing 选项列表。如下：

	typedef NS_ENUM(NSUInteger, EOCConnectionState) { 
            EOCConnectionStateDisconnected, 	
            EOCConnectionStateConnecting, 
            EOCConnectionStateConnected,	
    };	

    typedef NS_OPTIONS(NSUInteger, EOCPermittedDirection) {    
            EOCPermittedDirectionUp    = 1 << 0,
            EOCPermittedDirectionDown  = 1 << 1,    
            EOCPermittedDirectionLeft  = 1 << 2,    
            EOCPermittedDirectionRight = 1 << 3,	
    };

它的宏定义如下：

	#if (__cplusplus && __cplusplus >= 201103L &&        
          (__has_extension(cxx_strong_enums) ||         
           __has_feature(objc_fixed_enum))    
        ) ||
        (!__cplusplus && __has_feature(objc_fixed_enum))    
        #define NS_ENUM(_type, _name)            
                enum _name : _type _name; enum _name : _type    
        #if (__cplusplus)        
                #define NS_OPTIONS(_type, _name)
                _type _name; enum : _type
        #else
                #define NS_OPTIONS(_type, _name)
                enum _name : _type _name; enum _name : _type	
                #endif
    #else
        #define NS_ENUM(_type, _name) _type _name; enum    	
        #define NS_OPTIONS(_type, _name) _type _name; enum	
    #endif

宏定义有多个版本是因为有不同场景。先检查编译器是否支持新的枚举类型。这个检查使用了看起来相当复杂逻辑，但其实就是检查新功能是否是被支持。如果没有新功能，它会使用旧形式定义枚举。

如果，新功能存在，NS_ENUM 类型将如下展开：

    typedef enum EOCConnectionState : NSUInteger EOCConnectionState; 
    enum EOCConnectionState : NSUInteger {
         EOCConnectionStateDisconnected, 
         EOCConnectionStateConnecting,                          
         EOCConnectionStateConnected,
    };

NS_OPTIONS 宏根据是否是 C++ 编译器而使用另外不同定义方法。如果不是 C++ 编译器，它被展开成了 NS_ENUM 一样的形式。可是，如果是 C++ 编译器，它会被展开成不同的形式。为什么？因为 C++ 编译器在两个枚举值做位或时表现是不同的。位或如前所示，对于选项类型的枚举是常被用到的。当两个值作位或时，C++ 认为结果值是枚举中定义好的 NSInteger。对于枚举类型，隐式类型转换是不被允许的。为了展示这个，考虑 EOCPermittedDirection 被作为 NS_ENUM 展开会如何：

    typedef enum EOCPermittedDirection : int EOCPermittedDirection;
    enum EOCPermittedDirection : int {
        EOCPermittedDirectionUp    = 1 << 0,
        EOCPermittedDirectionDown  = 1 << 1,
        EOCPermittedDirectionLeft  = 1 << 2,
        EOCPermittedDirectionRight = 1 << 3,
    };

然后，考虑作如下事情：

    EOCPermittedDirection permittedDirections = EOCPermittedDirectionLeft | EOCPermittedDirectionUp;

如果编译器在 C++ 模式中（或可能在 Objective-C++ 中），这将导致如下错误：

    error: cannot initialize a variable of type
    'EOCPermittedDirection' with an rvalue of type 'int'

你将被要求对位或结果作显式的类型转换，转换为 EOCPermittedDirection。所以 NS_OPTIONS 枚举类型在 C++ 下定义稍稍不同以至于我们无法这么作。由于这个原因，如果你会对多个枚举值作位或运算，你必须总是使用 NS_OPTIONS。如果，你不会这么作，就使用 NS_ENUM。

一个枚举可以被使用在不同的场景下。选项和状态已经在前面展示了；但还有其它场景存在。错误码就是一个很好的例子。代替预定义或者常量，枚举提供了一个定义一组逻辑上相关联的状态码的方法。另外一个例子是风格。例如，如果你有一个 UI 元素可以以不同风格创建，那么枚举类型就是绝佳选择。

另外最后一点是关于在 switch 语句中使用枚举的。有时，你会作如下的事情：

    typedef NS_ENUM(NSUInteger, EOCConnectionState) { 
        EOCConnectionStateDisconnected, 
        EOCConnectionStateConnecting, 
        EOCConnectionStateConnected,
    };

    switch (_currentState) { 
       EOCConnectionStateDisconnected:
           // Handle disconnected state
           break; 
       EOCConnectionStateConnecting:
           // Handle connecting state
           break; 
       EOCConnectionStateConnected:
           // Handle connected state
           break;
    }

在 switch 语句中有一个缺省出口。可是，当用户在枚举类型上使用 switch 作状态机时，最好不要有缺省出口。理由是，如果你后面增添了一个状态，编译器将会提供一个警告，被添加的新的状态在 switch 中并没有被处理。如果有缺省出口，编译器将不会警告。这同样适用于使用 NS_ENUM 宏定义的枚举类型。例如，用于定义 UI 元素的风格，你常常会要确定 switch 语句是否处理了所有 UI 风格。

### 要点回顾

* 使用枚举给予状态机的状态，传递给方法的选项或者错误代码的值以可读的名字。

* 如果一个枚举类型定义了传递给方法的可以同时使用的选项，将其值定义为 2 的幂，以使得多个值可以通过位或操作同时使用。

* 使用 NS_ENUM 和 NS_OPTIONS 宏定义以显式类型定义的枚举类型。这么做意味着枚举成员的类型被确保为我们选择的类型，而不是编译器选择的类型。

* 在一个处理枚举类型的 switch 语句中不要实现 default case。这将有所帮助。因为，如果你添加了枚举项，编译器将警告 switch 没有处理所有的枚举值。