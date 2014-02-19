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

#### 要点回归
* Blocks 是 C，C++，和 Objective-C 的闭包。
* Blocks 可以带参数，也可以有返回值。
* Blocks 可以分配在栈，也可分配在堆，也可以是全局的。一个分配在栈的 block 可以被拷贝到堆，在堆中它可以被算作如标准 Objective-C 对象一样的引用。

## 条目 38 ： 为公共 Block 通过 typedefs 创建类型

#### 要点回归
* 使用类型定义使得 block 变量更易使用。
* 在定义新类型总是遵循命名规则，这样你就不会和其它类型发生冲突。
* 不要害怕为同一个 block 签名定义多个类型。你可能要通过改变 block 签名而不是类型，以重构某个使用了特定 block 类型的地方。

## 条目 39 ： 用 Blocks 回调减少代码分离

#### 要点回归
* 当有回调的业务逻辑需要根据对象的创建而申明时，使用 block 回调会很有用。
* Blocks 回调有可以直接和对象关联而不是和委托者关联的优点，委托常会要求交换对象，如果多个实例被观察着时。
* 当设计一个使用 blocks 回调的 API 时，考虑传递一个队列作为参数，设计一个闭包需要被排入的队列。 

## 条目 40 ： 避免 block 内强引用的对象引入的 Retain 循环

#### 要点回归
* 注意 blocks 使用了直接或间接 retain 了 block 的对象会引入 retian 循环这样的潜在问题。
* 确保 retain 循环在恰当的时候被打破，绝不要将这个任务留给 API 的使用者们。

## 条目 41 ： 推荐使用 dispatch queues 来同步锁
有时在 Objective-C 中，你会遇到一些麻烦的代码，因为它会被多个线程访问。这种情况通常需要应用通过锁的使用而有某种同步机制。在 GCD 之前，有两种方法完成同步，第一种是使用内建的同步块：

	- (void)synchronizedMethod {
		@synchronized(self) {
			// safe
		}
	}

这个结构自动创建基于给定对象的锁，并等待解锁直到它执行完包含在块中代码。在代码块的结尾，锁被释放。在这个例子中，被同步的对象是 self。这个结构通常是个好选择，因为它保证每个对象的实例可以相互独立的运行它们自己的 synchronizedMethod。可是，过度的使用 @synchronized(self) 可以引起低效的代码，因为每个同步块都将串行地执行所有这些块。如果，你过度地在 self 上使用同步，你最终会让代码在不相关地代码所持有地锁上做不必要的等待。

另外一种方法是直接使用 NSLock 对象：

	_lock = [[NSLock alloc] init];
	
	- (void)synchronizedMethod {
		[_lock lock];
		// Safe
		[_lock unlock];
	}
	
递归锁常通过 NSRecursiveLock 实现，它允许一个线程多次取出同一个锁而不会引发锁死。

这两种方法都不错但也有各自的缺点。例如，同步块能导致锁死在极端环境下，而且并不高效。直接使用锁，在进入锁死状态时，会很麻烦。

替代是使用 GCD，它用更简单更高效的方式提供锁。开发者使属性变为原子操作，是一个使用同步的好例子。这可以通过 atomic 属性参数（条目 6）完成。或者，如果访问器可以被手写，通常能看到如下代码：

	- (NSString*)someString { 
		@synchronized(self) {
			return _someString; 
		}
	}
	
	- (void)setSomeString:(NSString*)someString { 
		@synchronized(self) {
			_someString = someString; 
		}
	}

重复调用 @synchronized(self) 是很危险的如果滥用的话，因为所有这些块都会被彼此同步。如果多个属性这么做，每一个属性将会被和其它所有属性同步，这可能并不是你想要的。你真正想要的是每个属性的访问被独立同步。

顺便插一句，你应该警惕虽然这做了某些事来保证线程安全，它不能绝对确保对象的线程安全。所以，宁可使属性变为原子的。你被保证得到在使用属性时能获得合法的结果，但是如果你在同一个线程中多次调用取值器，你可能不能每次都得到相同的值。其它的线程可能在两次读操作之间已对属性做了写操作。

一个简单有效的替代是同步块或者对象锁用在串行线程中。将读和写分发到相同队列中以保证同步。这么做看起来会像这样：

	_syncQueue = dispatch_queue_create("com.effectiveobjectivec.syncQueue", NULL);
	
	- (NSString*)someString {
		__block NSString *localSomeString; 
		dispatch_sync(_syncQueue, ^{
			localSomeString = _someString; 
		});
		return localSomeString;
	}

	- (void)setSomeString:(NSString*)someString { 
		dispatch_sync(_syncQueue, ^{
			_someString = someString; 
		});
	}

这个某事背后的想法使所有对属性的访问操作都被同步了，因为取值器和赋值器所运行的 GCD 队列是同一个串行队列。撇开取值器中的 __block 关键字，已使得 block 可以给变量赋值（见 条目 37），这个方法非常简洁。所有的锁都在 GCD 中被处理，而 GCD 是在底层实现的，做过了很多的优化。因此，不用担心线程方面的事情而能够专注与编写你的访问器代码。

可是，我可以更进一步。赋值器无需是同步的。给实例变量赋值的 block 无需返回任何东西。这意味着你可以将赋值器改为如下样子：

	- (void)setSomeString:(NSString*)someString { 
		dispatch_async(_syncQueue, ^{
			_someString = someString; 
		});
	}

这个简单的从同步分发改变到异步分发提供了一个人在调用者看来更快的赋值器，但是读和写仍然被一个一个的串行执行。但是，有一个缺点，如果你做了检测，你可能会发现它有点慢；异步分发中，block 必须被拷贝。如果执行拷贝的时间和 block 的执行时间相当的话，它就会变慢。所以，在我们这个简单的例子中，它看起来是更慢了。可是，这个方法仍然是个好方法，如果将其理解为在 block 被分发执行很重的任务时潜在选项的话。

另外一个使这个方案更快速的方法是利用取值器可以彼此同时运行，但不可以和赋值器同时运行。这个方法 GCD 已有了。随后的代码无法使用同步块或者锁轻易实现。替代使用串行锁，考虑一下如果使用一个串行队列会发生什么：

	_syncQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	
	- (NSString*)someString {
		__block NSString *localSomeString; 
		dispatch_sync(_syncQueue, ^{
			localSomeString = _someString; 
		});
		return localSomeString; 
	}

	- (void)setSomeString:(NSString*)someString { 
		dispatch_async(_syncQueue, ^{
			_someString = someString; 
		});
	}

照此，这些代码不会同步工作。所有读和写都在同一个队列执行，但是这个队列是竞争的，读和写完全可能同时发生。这是我们在第一方案中就试图避免发生的。可是，一个简单的 GCD 功能，叫做障碍（barrier），可以解决这个问题。队列中有障碍 block 的函数形式如下：

	void dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
	void dispatch_barrier_sync(dispatch_queue_t queue, dispatch_block_t block);

一个障碍在队列中的执行和其它所有的 block 是互斥的。它们只和竞争队列有关，因为所有在串行队列中的 block
总是相互互斥执行的。当一个队列被执行了，下一个队列是障碍 block，队列将等待所有当前 block 完成，然后执行障碍 block。当障碍 block 结束执行，队列继续如常执行。

障碍可以被用于属性例子中的赋值器。如果赋值器使用障碍 block，属性读取仍然竞争执行，但是写操作会互斥执行。图 6.3 显示了有多个读操作和一个写草的队列。

![alt Queue](/images/blog/EffectiveObjC/6-3.jpeg "Concurrent Queue")

**图 6.3** 读为普通 block 和写为障碍 block 的竞争队列。读被竞争执行；写被互斥执行，就如它们像个障碍一样。

完成这个的代码很简单：

	_syncQueue =
	dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	
	- (NSString*)someString {
		__block NSString *localSomeString; 
		dispatch_sync(_syncQueue, ^{
			localSomeString = _someString; 
		});
		return localSomeString; 
	}

	- (void)setSomeString:(NSString*)someString { 
		dispatch_barrier_async(_syncQueue, ^{
			_someString = someString; 
		});
	}

如果你检测这个，你将会发现它快于使用串行队列。注意你可以在赋值器中使用同步障碍，和之前解释过的理由一样，这会更高效。测试每种方案选择一个对你的特别场景最好的方案是更谨慎的。

#### 要点回归
* 分发队列可以被用于提供同步语义，也就是说提供了一个 @synchronized 块或者 NSLock 对象的替代品。
* 混合同步和异步分发可以提供和普通的锁效果相同的同步行为，却不会堵塞异步分发中的线程调用。
* 竞争队列和障碍 block 可以用于使同步行为更高效。

## 条目 42 ： 推荐 GCD 而不是 performSelector 系列方法
由于 Objective-C 的极为动态的特性（条目 11），一些定义在 NSObject 中的方法允许你跳用任何你想调用的方法。它们允许你推迟方法调用的执行，或者推迟决定它需要在哪个线程上运行。它们曾是十分有用的功能；可是现在，诸如 Grand Central Dispatch 和 block 这样的技术使它们变得不再重要。虽然你仍然可以看到使用它们的代码，但我鼓励你清除它们。

这个方法族里最基本的使 performSelector:。它带一个参数，就是要执行的选择器，签名如下：

	- (id)performSelector:(SEL)selector

这和直接调用选择器是等价的。所以，以下两行代码是等价的：

	[object performSelector:@selector(selectorName)];
	[object selectName];
	
它可能看起来有点累赘。如果它是使用方法的唯一途径，那确实有点累赘。可是它真正的力量来自选择器可以在运行时决定。这样的动态绑定意味这你可以这么做：

	SEL selector;
	if (/* some condition */) {
		selector = @selector(foo);
	}
	else if (/* some condiftion */) {
		selector = @selector(bar);
	}
	else {
		selector = @selector(baz);
	}
	[object performSelector:selector];
	
这个代码相当灵活，并常可以被用于简化复杂代码。另外一种用法是存储一个在某个事件发生后必须执行的选择器。无论哪种情况，编译器都直到运行时才直到哪个选择器会被执行。。但是这个功能的代价是，如果你在 ARC 下编译，编译器会抛出如下警告：

	warning: performSelector may cause a leak because its selector is unknown [-Warc-performSelector-leaks]

你可能不希望这样！如果你确实不希望这样，你可能知道为什么你应该对这些方法小心。这个消息对你来说，看起来可能有些奇怪，你会想知道为何提到了泄漏。毕竟，你只是简单的试图调用一个方法。原因是编译器不知道哪个选择器将会被调用，因此不知道方法签名，返回类型，甚至不知道是否有返回值。编译器也不知道方法名，因此不能使用 ARC 内存管理规则就地哪些返回值应该被释放。为了这个原因，ARC 为了安全起见不会释放。可是，结果可能会造成泄漏，因为返回对象可能被保留过。

考虑如下代码：

	SEL selector;
	if ( /* some condition */ ) {
		selector = @selector(newObject); 
	} else if ( /* some other condition */ ) {
		selector = @selector(copy); 
	} else {
		selector = @selector(someProperty); 
	}
	id ret = [object performSelector:selector];
	
这个例子对前例做了细小的变化以展示出问题。在前两个选择器的情况下， ret 对象将需要被代码释放；而第三个选择器，则无需释放。不仅仅在 ARC 环境下，在严格遵守了方法命名指导的非 ARC 环境下，也会是如此。没有 ARC （因此没有编译器警告），ret 对象将需要被释放如果第一或者第二个条件为真时，但其它情况下则无需释放。这很容易被忽视，甚至静态分析器都不能帮助检查出随后的内存泄漏。这是一个慎重使用 performSelect 系列方法的理由。

另外一个这些方法并不理想的理由是返回值只能是 void 或者对象类型。 performSelector 方法的返回值是 id，但选择器返回 void 也是合法的。虽然复杂的类型转换可以使用返回为其它诸如整型，浮点型的选择器，但这会很脆弱。技术上返回任何与指针大小相同的类型都是可能的，因为 id 类型是一个指向任意 Objective-C 对象的指针：在 32 位构架中，就是任何 32 位宽类型；在 64 位构架中，就是任何 64 位宽类型。如果返回值是一个 C 结构体，就不能使用 performSelector 方法。

两个 performSelector 变种可以传递参数，其定义如下：

	- (id)performSelector:(SEL)selector
			   withObject:(id)object
	- (id)performSelector:(SEL)selector
			   withObject:(id)objectA
			   withObject:(id)objectB

例如，这些变种可以用于在一个对象上为一个叫 value 属性赋值：

	id object = /* an object with a property called 'value' */; 
	id newValue = /* new value for the property */; 
	[object performSelector:@selector(setValue:)
				 withObject:newValue];

这些方法可能看起来有用，但是它们有着严重的缺陷。被传入的参数必须是对象，因为它的类型永远是 id。所以如果一个选择器需要整数，或浮点数作参数，那么这些方法就不能使用。另外，选择器最多可以有两个参数，使用 performSelect:withObject:withObject: 。没有执行多于两个参数的选择器的等价方法。

performSelector 系列方法的另外个功能是选择器可以被延迟或者在另外一个线程被执行。一些这类方法如下：

	- (void)performSelector:(SEL)selector 
				 withObject:(id)argument
				 afterDelay:(NSTimeInterval)delay 

	- (void)performSelector:(SEL)selector
				   onThread:(NSThread*)thread
				   withObject:(id)argument
				   waitUntilDone:(BOOL)wait

	- (void)performSelectorOnMainThread:(SEL)selector
							  withObject:(id)argument 
						   waitUntilDone:(BOOL)wait

可是，这些方法是有制约。例如，没有方法可以延迟执行有两个参数的选择器。基于同样的原因，线程方法也不是通用的。要使用这些方法的代码常常会将多个参数打包入一个字典，并在被调用方法中解包，因此这会增加开销和潜在的 bug。

所有这些限制都可以使用一个替代物解决。主要替代是使用 blocks （条目 37）。更进一步，blocks 配合 Grand Center Dispatch （GCD）使得你可以解决所有因线程关系而使用 performSelect 方法的情况。延迟执行可以通过 dispatch_after 完成，在另外一个线程执行可以通过 dispath_sync 和 dispath_async 完成。

例如，延迟执行任务，你应该优先使用后者：

	// Using performSelector:withObject:afterDelay:
	[self performSelector:@selector(doSomething) 
			   withObject:nil
			   afterDelay:5.0];

	// Using dispatch_after
	dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0 * NSEC_PER_SEC)); 
	dispatch_after(time, dispatch_get_main_queue(), ^(void){
		[self doSomething]; }
	);

在主线程中执行任务：

	// Using performSelectorOnMainThread:withObject:waitUntilDone:
	[self performSelectorOnMainThread:@selector(doSomething) 
						    withObject:nil
	  	 		      	 waitUntilDone:NO];

	// Using dispatch_async
	// (or if waitUntilDone is YES, then dispatch_sync) 
	dispatch_async(dispatch_get_main_queue(), ^{
		[self doSomething]; 
	});

#### 要点回归
* performSelector 系列方法有潜在得内存管理问题。如果没有办法决定哪个选择器将被执行，ARC 编译器将不能插入正确得内存管理调用。
* 系列方法有相当限制，无法处理返回值，以及可以发送给方法的参数数量。
* 允许在不同线程里执行 performSelector 系列方法的，最好使用特定的 Grand Central Dispatch（GCD）调用 block 来代替。

## 条目 43 ： 知道何时使用 GCD 和 何时使用操作队列

#### 要点回归
* 分发队列不是任务管理和多线程的唯一解决方案。
* 操作队列提供了高级别的，Objective-C API，并能完成大部分 GCD 的工作。这些队列和 GCD 代码配合，也可以完成更为复杂的事情。

## 条目 44 ： 使用 dispatch group 以利用平台可扩张性

有时在 Objective-C 中，你会遇到一些有点麻烦的代码，因为它会被多个线程访问。这种情况通常要求应用通过锁的使用而有某种同步机制。

#### 要点回归
* Dispatch groups 被用于组织一个任务集合。你可以选择在任务组执行完成时，接到通知。
* Dispatch groups 被用于在一个竞争的分发队列中执行多个任务。在这种情况下，GCD 基于系统资源，处理同一时期执行的多任务的排期。这些工作如果自己来写，将需要大量代码。

## 条目 45 ： 使用 dispatch_once 执行有线程安全要求的代码

#### 要点回归
* 线程安全的单一的代码执行是一个普遍任务。GCD 通过 dispatch_once 提供了一个简单易用的工具。
* Token 应该被申明为 static 或者 global 这样使得传入每个需要单一执行的 block 的 token 都是一样的。

## 条目 46 ： 避免 dispatch_get_current_queue

#### 要点回归
* dispatch_get_current_queue 方法通常并不会按照你期望的那样执行。它已经被废弃了，现在只应该用于调试。
* 分发队列被按层级结构组织；因此，当前队列不能被简单地通过单个队列对象描述。
* 队列特定数据可以用于解决使用 dispatch_get_current_queue 的问题，它能避免由于非重入代码的死锁。
 