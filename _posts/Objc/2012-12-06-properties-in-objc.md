---
layout:    post
title:     Objective-C 中使用属性 (Properties)
category:  blog
description: iPhone 上的 Objective-C 中如何与内存...
tags: ObjC
---

在 iPhone 中如何使用 Objective-C 与内存打交道这个主题由一个包含三部分的系列文章组成，本篇是其中的第三篇。

在系列文章的[第一篇]()中，我们覆盖了 Objective-C 中如何使用实例变量和引用计数管理内存。

在系列文章的[第二篇]()中，我们覆盖了如何使用 Instruments 和其他辅助工具检查自己的应用的内存泄漏和内存相关错误。

在系列文章的第三篇也是最后一篇中，我们将讨论一下属性和 Objective-C。 我们将讨论属性是什么，他们如何工作，和一些可以用于避免内存相关问题的简单规则。

如果你还没有下载［示例项目]()，请下载它。此示例项目是我们在本系列文章中逐步建立起来的，现在我们用它作为本篇的一个起点。

##Retain Your Memory

让我们花一点时间回顾一下我们目前在内存管理方面的项目。在 RootViewController 中有两个实例变量: _sushiTypes， 和 _lastSushiSelected 。

	@interface RootViewController : UITableViewController {
		NSArray * _sushiTypes;
		NSString * _lastSushiSelected;
	}
	@end

对于 _sushiTypes, 数组在 viewDidLoad 中通过 alloc/init 创建，在 viewDidUnload 和 dealloc 中被释放：

	//In viewDidLoad.  Afterwards, retain count is 1.
	_sushiTypes = [[NSArray alloc] initWithObjects:@"California Roll",
					@"Tuna Roll", @"Salmon Roll", @"Unagi Roll", 
					@"Philadelphia Roll", @"Rainbow Roll",
					@"Vegetable Roll", @"Spider Roll",
					@"Shrimp Tempura Roll", @"Cucumber Roll",
					@"Yellowtail Roll", @"Spicy Tuna Roll",
					@"Avocado Roll", @"Scallop Roll", nil];
 
	// In viewDidUnload and dealloc.  Afterwards, retain count is 0.
	[_sushiTypes release];
	_sushiTypes = nil; 

对于 _lastSushiSelected，当用户在列表中选择一行时它被赋值，它要么在被赋值之前被释放，要么在 dealloc 中释放。

	//In tableView:didSelectRowAtIndexPath.  
	//Releases any existing object first and then retains the new object.
	[_lastSushiSelected release];
	_lastSushiSelected = [sushiString retain];
 
	//In dealloc
	[_sushiTypes release];
	_sushiTypes = nil;

这个方式肯定是有效的，但是他要求你每次给变量赋值时都细致考虑内存问题。那么，让我们看看是否有简单点的方法！

##Get 很简单，Set 要写点代码

如果你对其他语言如 Java 或者 C# 很熟悉，你可能也很熟悉取值器(getters)和赋值器(setters)的概念。

当你有了一个实例变量如 _sushiTypes，你通常也希望别的类可以访问到它。但是允许别的类直接访问实例变量一般说来都是不好的做法，因为它破坏了代码的封闭性。

所以作为代替，你可以生成一个叫 “getSushiType” 的方法(或可能仅仅叫 “SushiTypes” 易节省三个字符) 和一个叫 “setShushiTypes” 的方法。这样更好一些因为之后你可以改变实例变量的名字而不破坏其他类，或者在方法中添加额外的功能(如可以记录下任何时候任何人试图获取这个变量的行为)。

如此加入取值器和赋值器不仅仅有利于那些试图使用这些变量的外部类，它也是把内存管理代码统一起来的便捷方法。让我们看看如何为这两个变量添加取值器和赋值器。

首先在 RootViewController.h 的底部加入取值器和赋值器的函数签名：

	- (NSArray *)sushiTypes;
	- (void)setSushiTypes:(NSArray *)sushiTypes;
	- (NSString *)lastSushiSelected;
	- (void)setLastSushiSelected:(NSString *)lastSushiSelected;

然后在 RootViewController.h 的底部加入函数实现：

	- (NSArray *)sushiTypes {
    	return _sushiTypes;
	}

	- (void)setSushiTypes:(NSArray *)sushiTypes {
    	[sushiTypes retain];
    	[_sushiTypes release];
    	_sushiTypes = sushiTypes;
	}

	- (NSString *)lastSushiSelected {
    	return _lastSushiSelected;
	}
 
	- (void)setLastSushiSelected:(NSString *)lastSushiSelected {
    	[lastSushiSelected retain];
    	[_lastSushiSelected release];
    	_lastSushiSelected = lastSushiSelected;
	}

取值器不言自明 - 他们仅仅返回实例变量。

赋值器增加了传入参数的引用计数，减少了旧实例变量的引用计数，并且将实例变量设置为传入参数。通过这种方法，在实例变量复制时对象正确地拥有存于实例变量中对象的引用计数。

你可能想知道为什么赋值器依照顺序先调用 retain/release 然后再给实例变量赋值。依照这个顺序是为了避免在传入参数和实例变量是同一个值时可能产生的错误。如果不确定为什么，可以脑袋里过一下这个过程，想一想将发生什么。

注意命名实例变量以下划线开始(而不是什么也没有)使得写这类方法方便很多。如果你已经命名了一个实例变量为“sushiTypes”，我们无法将 setSushiTypes 的参数命名为 “sushiTypes” 因为名字将会又冲突。这也是一个很好的简单的方法告知别人，你什么时候是在使用一个实例变量，什么时候不是。

最终，注意这些取值器和赋值器不是线程安全的。但是这对这个应用而言并不是问题，取值器/赋值器将只被主线程访问。

##现在可以 Get/Set 了

现在你有了新的取值器和赋值器了，修改类的余下代码使用它们。让我们从 sushiTypes：

	// In viewDidLoad
	self.sushiTypes = [[[NSArray alloc] initWithObjects:@"California Roll", 
    	           @"Tuna Roll", @"Salmon Roll", @"Unagi Roll",
    	           @"Philadelphia Roll", @"Rainbow Roll",
            	   @"Vegetable Roll", @"Spider Roll", 
            	   @"Shrimp Tempura Roll", @"Cucumber Roll",
            	   @"Yellowtail Roll", @"Spicy Tuna Roll",
            	   @"Avocado Roll", @"Scallop Roll", nil] autorelease];
 
	// In viewDidUnload and dealloc
	self.sushiTypes = nil;

调用 “self.sushiTypes = xxx” 和调用 “[self setSushiTypes:xxx]” - 点号对于我而言更好看一些。

所以基本上，我们现在用赋值器替代直接给 _sushiTypes 这个实例变量直接赋值。调用赋值器将为你所传的对象的引用计数加1。所以，我们不能仅传 alloc/init 的结果(因为，这样引用计数将为2，这是不正确的)－我们在结尾调用 autorelease。

在 viewDidUnload 和 dealloc 中，不用先调用 release 然后手动置值为 nil，我们可以直接调用赋值器。如果，你将 nil 带入 setSushiTypes，你将看当为何可以如此做：

	[nil retain];  // Does nothing
	[_sushiTypes release];
	_sushiTypes = nil;

顺带说一句，因为可能你已察觉到－一些人说，“不要在 dealloc 或者 init 中使用取值器和赋值器”。他们这么说是因为取值器和赋值器是随意的函数，它们可能有副作用并破坏一些东西。但是恕我直言－如果你编写了它们并且知道他们没有问题－那么在可以简化代码的情况下就使用他们。

现在更改代码使用 lastSushiSelected 的赋值器：

	// In tableView:didSelectRowAtIndexPath,
	// delete the two lines about 	_lastSushiSelected and replace with:
	self.lastSushiSelected = sushiString;
 
	// In dealloc
	self.lastSushiSelected = nil;

Wow - 对于内存的担心少了很多，不是吗？赋值器包含了所有要认真处理内存管理事务的代码。

##一个简单的提议

所以写取值器和赋值器对于别的类访问你的实例变量是很有用的，并且有时也使内存管理更为简单。

但是一遍一遍编写这些方法实在是件很蛋痛的事情！没有人希望一遍一遍做相同的事情，所以 Objective-C 有一个很有用的功能来帮助我们解脱：属性(properties)。

你可以用很少几行代替所有取值器/赋值器代码！自己试一下。在 RootViewController.h，去除取值器和赋值器的原型用以下代码代替：

	@property (nonatomic, retain) NSArray * sushiTypes;
	@property (nonatomic, retain) NSString * lastSushiSelected;

这是使用属性的第一步：创建属性声明。

属性声明以关键字 @property 开始。然后在括号里你可以放属性的描述(将在后面讨论)。最后，你给出类型和属性的名字。

属性的描述是几个特殊的关键字，用以告诉编译器如何生成取值器和赋值器。这里，你指明了两个描述：nonatomic，它告诉编译器不要担心多线程，retain，它告诉编译器在给实例变量赋值之前先在 传入值上调用 retian。

在其他情况下，你可能需要使用 “assign” 这个属性描述替代 retain，它告诉编译器不要在传入值上调用 retain。或者也许是 “copy” 这个描述，它告诉编译器在赋值之前制作传入值的一个拷贝。

好了！为了结束关于属性的使用这一部分，转到 RootViewController.m，删去你之前写的取值器和赋值器，添加如下代码在文件顶部：

	@synthesize sushiTypes = _sushiTypes;
	@synthesize lastSushiSelected = _lastSushiSelected;

这两行指示编译器基于你在头文件中加的属性定义自动生成取值器和赋值器代码。你以关键字 @synthesize 开始，然后给出属性名，然后(如果有不同的名字)告诉它需关联哪个实例变量。

这就完了！编译运行代码，它应该运行如初。但是如果你比较现在的代码和开始的代码，我想你会同意现在的代码更易懂一些，可以避免更多的错误。

你已学会了属性，和他们如何工作。

##综合的策略

我想用一些我有时会在 Objective－C 中用到的简化内存管理的策略来总结一下这篇文章。

如果你遵守这些规则，大部分时候，你都可以远离内存相关的麻烦。当然，盲目遵循这些规则并不能替代理解其中的原理，但是它可以使事情变得简单并帮助你避免错误，特别当你还是初学者的时候。

我将先列出这些规则，然后逐条讨论。
<ol>
<li>总是为实例变量生成属性。</li>
<li>如果使类，标记为 retain。如果不是，标记为 assign。</li>
<li>无论何时创建一个变量，都使用 alloc/init/autorelease 语式。</li>
<li>无路何时给一个变量赋值，总是使用 “self.xxx = yyy” (换句话说，使用属性)。</li>
<li>对于你的每一个实例变量，在 dealloc 中调用 "self.xxx = nil"。如果它是 outlet 或者是你在 viewDidLoad 创建的东西，在 viewDidUnLoad 中调用 "self.xxx = ni"。</li>
</ol>
好，现在进入讨论！

对于规则1：为每个实例变量都生成属性，你可以让编译器为你写内存管理代码。缺陷是不能保持私有数据的封装，一不小心就导致更多相互关联的代码。

对于规则2：无论何时赋值，都向变量发送 retain 消息。这样，你可以确保你任何时候都可以访问你的实例变量。

对于规则3：当你创建变量时，使用 alloc/init/autorelease 语式 (就像你之前在创建 NSArray 类型的 sushiTypes 时看到的那样)。这种方法，内存将在晚些时候自动释放。如果你要长期保持变量，你应该将它赋给一个属性，将它放入数组，或者增加它的引用计数。

对于规则4：无论何时赋值都使用 self.xxx 语法(即使用属性)。这样，你将通过属性确保释放了旧的变量并保留了新变量。注意某些程序对于在构造器和析构器(即 dealloc)中使用取值器和赋值器，表示担忧，但是我认为这并不是什么值得担忧的事情如果你写了并理解那些代码。

过对于规则5：调用“self.xxx = nil” 将调用你的属性代码，并为引用计数减一。不要忘记在 viewDidUnload 中做同样的事情！

##关于 Cocos2D 的简化的策略

这个 blog 中有很多狂热的 Cocos2D 粉丝，所以我为你们这些人加了特别的一节。

上面的那些策略对于 Cocos2D 而言通常是矫枉过正，因为大多数时候你的实例变量仅仅是一些你要在你的当前层使用的对象的引用。只要这些对象当前在层上，Cocos2D 就会保留他们。

所以为了避免生产不必要的属性，这里有我如何使用 Cocos2D:

<ol>
<li>完全不要使用属性。</li>
<li>将你创建的对象直接赋给实例变量。</li>
<li>因为他们将成为当前层的子对象，只要他们上了当前层，Cocos2D 就将增加他们的引用计数。</li>
<li>在你从层中去除对象时，将当前对象置为 nil。</li>
</ol>

我仅是个人认为这样的开发方式更为简单快速。

注意如果你的对象当前不在层上，这些策略将不适用－你需要适用其他的策略。

##接下来去哪儿？

这儿是上面的文章中使用到的[实例项目](http://)，已经改用了属性。

如果你有任何关于属性和内存管理的问题，欢迎在后面的论坛留言！同时欢迎分享如果你有任何其他内存管理的策略，技巧，或者对其他开发者有用的建议。

关于内存管理的系列文章，希望它对拨开 Objective-C 的内存管理的神秘面纱有帮助。
 
---

**原文连接**：[http://www.raywenderlich.com/2712/using-properties-in-objective-c-tutorial](http://www.raywenderlich.com/2712/using-properties-in-objective-c-tutorial)
