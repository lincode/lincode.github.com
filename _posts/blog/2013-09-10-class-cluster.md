---
layout:    post
title:     类簇 Class Clusters
category:  blog
description: Objective-C 的一种设计模式：类簇
tags: ObjC
---

##类簇 Class Clusters
类簇是 Foundation 库中广泛使用的一种设计模式。类簇将一组私有的具体的子类聚于一个公开的抽象父类之下。这种类的组合方式使得一个面向对象库的公共可见结构简化了，同时又不会削减其功能。类簇是基于 **抽象方法** 这种模式。


##无类簇：简单概念，复杂接口
为了展示类簇结构和它的便利，考虑这样一个问题，构建定义了存储不同类型数字(char, int, float, double)的对象的类结构。因为不同类型有很多的共同功能 (例如，它们可以从一种类型转换到另外一种类型，可以用字符串表示)，它们可以用一个类来表示。可是，它们的存储需求是不同的，所以，将它们全用一种类表示又不是有效率的。考虑到这种因素，我们可以设计一个如图 1-1 描绘的类结构来解决这个问题。

![alt cluster1](/images/blog/Clusters/cluster1.gif "A simple hierarchy for number class")

**Number** 是一个抽象的父类，为它的子类声明了那些共同操作的方法。可是，它没有声明实例变量以存储数字。子类声明了实例变量，并通过 **Number** 的接口暴露出去。

![alt cluster2](/images/blog/Clusters/cluster2.gif "A more complete number class hierarchy")

简单的概念－－创建类用于存储数字值－－能简单生出十几个类。类簇结构则展示了一种比较此概念更为简单的设计。

##有类簇：简单概念，简单接口
使用类簇将产生 图 1-3 (私有类为灰色) 所示类结构。

![alt cluster3](/images/blog/Clusters/cluster3.gif " Class cluster architecture applied to number classes")

这个结构的使用者只能看到公有类， **Number**，那么如何为各个子类的分配实例呢？答案是通过抽象父类处理实例化。


##创建实例
类簇中的抽象父类必须声明创建它的私有子类实例的方法。这是父类的职责，各个子类对象基于你调用父类的创建方法来获得，你不可，也不能选择该方法创建出来的实例的类型。

在 Foundation 库中，你通常会通过调用 className… 或者通过 alloc… 和 init… 方法创建对象。拿 Foundation 库的 NSNumber 做例子，你可以向 number 对象发送如下消息：

	NSNumber *aChar = [NSNumber numberWithChar:’a’];
	NSNumber *anInt = [NSNumber numberWithInt:1];
	NSNumber *aFloat = [NSNumber numberWithFloat:1.0];
	NSNumber *aDouble = [NSNumber numberWithDouble:1.0];

你不用为释放工厂方法返回的对象负责。很多类都提供标准的 alloc… 和 init… 方法来创建对象，并要求你管理对象销毁。

每个返回的对象－－aChar, anInt, aFloat 和 aDouble －－可能属于不同的私有子类(实际上确实这样)。虽然，每个类的对象的类别是隐藏的，但，由于接口是由抽象父类 NSNumber 声明的，所以它们的接口是公开的。简化地认为 aChar，anInt，aFloat 和 aDouble 对象是一个 NSNumber 类别的对象，有点不太精确。实际上，它们是由 NSNumber 类的方法创建，并通过由 NSNumber 类的方法访问的。


##有多个公开父类的类簇 
在上面的例子中，一个抽象公共父类为多个私有子类声明了接口。这是一个纯粹意义上的类簇。有两个或两个以上的抽象公共父类为类簇声明接口也是可能的，也经常是需要的。很明显，在 Foundation 库中，有这样的例子：


* NSData  ：		NSData, NSMutableData
* NSArray  ：		NSArray, NSMutableArray
* NSDictionary	：  NSDictionray, NSMutableDictionary
* NSString		：  NSStrring, NSMutableString

还有这种类型的其他类簇存在，但是这些类簇清楚的展示了两个抽象类合作为一个类簇声明程序接口。在每一个类簇中，一个公共类声明所有类簇对象都会实现的方法，其余的公共类声明仅为适当的类簇对象改变其内容的方法。

这种类簇的组合有助于使一个面向对象库的程序接口更居表达性。例如，想象一个代表一本书的对象，声明了如下方法：
	
	- (NSString *)title;

书对象可以返回他自己的实例对象，或者创建一个新的字符串对象同时返回它－－这都没有关系。返回的字符串不可改变是清楚无疑的。所有试图改变返回对象的尝试都会引出一个编译错误。


##在类簇中创建子类
类簇结构包含简单性和可扩展性的折衷：拥有少量公共类而隐藏了大量的私有子类，使得学习和使用库里的类更为简单，但有时创建在类簇中创建子类则更为困难了。可是，如果创建子类是一个少有的需求，那么很清楚，类簇结构是有利的。类簇被使用在 Foundation 库中，就是这种情况。

如果，你找到一个类簇不能提供你需要的功能，那么可以考虑子类。例如，想象一下你需要创建一个数组对象，它的存储是基于文件而不是象 NSArray 一样基于内存的。因为你改变类存储的相关机制，你需要创建一个子类。

另一方面，在某些情况下，定一个一个包裹了类簇对象的类可能是更为高效和简单的选择。例如，我们需要在任何数据发生了改变时，发出警告。这种情况下，创建一个简单包裹了 Foundation 库定义的数据对象的类可能会是最佳方法。发往对象要求修改数据的消息是可以被干预的，截获消息，在消息做动作，再将他们分发给被包裹对象。

总而言之，如果你需要管理你的类的存储，创建一个正真的子类。否则，创建一个组合对象，在一个你自己设计的对象中潜入一个标准 Foundation 库对象。下面两章给出关于这两种方法的更多细节。

### 一个真正子类
在一个类簇中，创建新类需要：

* 必须为类簇抽象父类的子类
* 声明其自己的存储数据用的实例变量
* 重写父类的所有实例化方法
* 重写父类的原始方法 (如下面描述)

因为，类簇的抽象父类在类簇结构中是唯一暴露的公共结点，第一点是显而易见的。这隐含了新类必须继承类簇的接口，但不继承实例变量，因为抽象父类不声明实例变量。因此，需要第二点，子类必须声明实例变量。最后，子类必须重写所有它继承的直接访问实例变量的方法。这类方法叫做：原始方法。

一个类的原始方法古城其接口的基础。拿 NSArray 类为例，它声明了一些管理对象数组的接口。在概念上，一个数组保存了一些可通过索引访问的数据。NSArray 通过两个原始方法 count 和 objectAtIndex: 表达来了这个抽象概念。以这些方法为基础，其余的方法－－衍生方法－－都可以被实现。

两个衍生方法和它们可能的实现：

* lastObject	通过发送如下消息给数组对象，找到最后一个对象 [self objectAtIndex:([]self count] - 1)]
* containsObject:	通过 objectAtIndex 方法遍历数组来测试数组是否包含某个对象

将一个接口切分为原始方法和衍生方法，使得创建子类容易了很多。你的子类必须重写被继承的原始方法，但是所有衍生方法都可以正常运行而无需改变。

原始－衍生方法的区分适用于一个完全实例化的对象的接口。在子类中 init… 方法如何处理的问题同时也必须解决。

通常，一个类簇的父类声明了一些 init… 和 +className 方法。如同在 [创建实例](#创建实例) 中描述的，抽象类基于你选择了哪个 init… 或者 +className 方法决定哪个具体实例要被实例化。你可以考虑抽象类为便利地适用子类声明了这些方法。因为抽象类没有实例变量，它无需实例方法。

你地子类必须声明自己的 init… (如果它需要实例化自己的实例变量的话)，并且声明可能的 +className 方法。这并不取决于它继承了什么。为了维持实例化链的联系，它必须在自己的实例化创建器中，调用父类的默认实例化创建器。它也必须重写所有其他继承下来的实例化方法，并以合理的方法实现它们的行为([多个实例化创建器和默认实例化创建器](#https://developer.apple.com/library/ios/documentation/general/conceptual/CocoaEncyclopedia/Initialization/Initialization.html#//apple_ref/doc/uid/TP40010810-CH6-SW3) 讨论了默认实例化创建器)。在类簇中，抽象父类的默认实例化创建器总是 init。

### 真正的子类：例子
让我们假设，你要创建一个 NSArray 的子类，名为 MonthArray。它可以返回给定索引位置的月份名。可是，一个 MontyArray 对象不能将一个月份名字的数组作为实例变量存储。

	#import <foundation/foundation.h>
	@interface MonthArray : NSArray
	{
	}
 
	+ monthArray;
	- (unsigned)count;
	- (id)objectAtIndex:(unsigned)index;
 
	@end

注意 MonthArray 类并没有声明 init… 方法，因为它没有实例变量需要实例化。count 和 objectAtIndex: 方法就是所有的被继承的原始方法了。

MonthArray 的实现如下：

	#import "MonthArray.h"
 	
	@implementation MonthArray
 	
	static MonthArray *sharedMonthArray = nil;
	static NSString *months[] = { @"January", @"February", @"March",
    	@"April", @"May", @"June", @"July", @"August", @"September",
    	@"October", @"November", @"December" };
 	
	+ monthArray
	{
    	if (!sharedMonthArray) {
        	sharedMonthArray = [[MonthArray alloc] init];
    	}
    	return sharedMonthArray;
	}
 	
	- (unsigned)count
	{
 		return 12;
	}
 
	- objectAtIndex:(unsigned)index
	{
    	if (index >= [self count])
        	[NSException raise:NSRangeException format:@"***%s: index
            	(%d) beyond bounds (%d)", sel_getName(_cmd), index,
            	[self count] - 1];
	    else
    	    return months[index];
	}
 
	@end

因为 MonthArray 重写了被继承的原始方法，它继承的衍生方法无需重写也可以正确工作。NSArray 的 lastObject，containObject:，sortedArrayUsingSelector:，objectEnumetor，和其他方法都可以为 MonthArray 对象正常工作。

### 组合对象
通过在一个你设计的对象中嵌入一个私有的类簇对象，你可以创建一个组合对象。这个组合对象仅依靠类簇对象实现基本功能，组合对象截获消息后通过特殊的方发做处理。这个结构减少了你必须编写的代码量，并让你充分利用了 Foundation 库中的测试充分的代码。图 1-4 描述了这个结构。

![alt compositeobject](/images/blog/Clusters/compositeobject.gif " Class cluster architecture applied to number classes")

组合对象必须将自己声明为类簇的抽象父类的子类。作为子类，它必须重写父类的原始方法。它也可以重写衍生方法，但是这不是必须的，因为，衍生方法是基于原始方法实现的。

NSArray 的 count 方法作为例。重写函数可以简单地如下实现：

	- (unsigned)count {
    	return [embeddedObject count];
	}
	
然而，你的对象在重载方法中放入按自己目的实现的代码。	

### 组合对象：例子
为了展示组合对象的适用，想象一下你要一个可变数组对象，它可以在改变数组内容时测试改变是否符合一些规则。下面的例子描述了一个名为 ValidatingArray 的类，它包含一个标准的可变数组对象。ValidatingArray 重写了所有的在父类 NSArray 和 NSMutableArray 中声明的原始方法。它也声明了 ValidatingArray 和 init 方法，用于创建和初始化实例：

	#import <foundation/foundation.h>
 	
	@interface ValidatingArray : NSMutableArray
	{
    	NSMutableArray *embeddedArray;
	}
 	
	+ validatingArray;
	- init;
	- (unsigned)count;
	- objectAtIndex:(unsigned)index;
	- (void)addObject:object;
	- (void)replaceObjectAtIndex:(unsigned)index withObject:object;
	- (void)removeLastObject;
	- (void)insertObject:object atIndex:(unsigned)index;
	- (void)removeObjectAtIndex:(unsigned)index;
 	
	@end

实现文件显示在 ValidatingClass 类的 init 方法中，嵌入对象被创建并且被赋给 embedddedArray 对象。显示了简单地访问数组，不改变其内容的消息是基于嵌入对象的。改变其内容的消息是经过审查的(通过伪码实现)，只有在通过了假想的测试情况下才和嵌入对象有关。


	#import "ValidatingArray.h"
 
	@implementation ValidatingArray
 
	- init
	{
    	self = [super init];
    	if (self) {
        	embeddedArray = [[NSMutableArray allocWithZone:[self zone]] init];
	    }
    	return self;
	}
 
	+ validatingArray
	{
    	return [[[self alloc] init] autorelease];
	}
 
	- (unsigned)count
	{
    	return [embeddedArray count];
	}
 	
	- objectAtIndex:(unsigned)index
	{
    	return [embeddedArray objectAtIndex:index];
	}
 
	- (void)addObject:object
	{
    	if (/* modification is valid */) {
        	[embeddedArray addObject:object];
    	}
	}
 
	- (void)replaceObjectAtIndex:(unsigned)index withObject:object;
	{
    	if (/* modification is valid */) {
        	[embeddedArray replaceObjectAtIndex:index withObject:object];
    	}
	}
 
	- (void)removeLastObject;
	{
	    if (/* modification is valid */) {
        	[embeddedArray removeLastObject];
    	}
	}

	- (void)insertObject:object atIndex:(unsigned)index;
	{
    	if (/* modification is valid */) {
        	[embeddedArray insertObject:object atIndex:index];
	    }
	}

	- (void)removeObjectAtIndex:(unsigned)index;
	{
    	if (/* modification is valid */) {
        	[embeddedArray removeObjectAtIndex:index];
    	}
	}



##来源

---

**原文链接**：[链接](https://developer.apple.com/library/ios/documentation/general/conceptual/CocoaEncyclopedia/ClassClusters/ClassClusters.html)
