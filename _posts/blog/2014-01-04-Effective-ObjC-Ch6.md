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

Blocks 提供了闭包。这个语音功能被作为 GCC 编译器的一个扩展加入，在所有现代 Clang 版本中都是可用的（Clang 是 Mac OS X 和 iOS 开发所使用的编译器项目）。blocks 可以正确运行的运行时环境从 Mac OS X 10.4 和 iOS 4.0 开始便具备了。这个语音特性技术上讲是一个 C 级别的功能因此可以被用于 C，C++，Objective-C 和 Objective-C++ 代码，被支持的编译器所变异，运行于支持运行 block 运行时环境。

#### Block 基础
一个 block 类似一个函数但相对于其他函数是内联定义，并且共享了它定义处作用域的上下文。用于指示一个 block 的愈发标识是脱字符，后面跟着包含了 block 实现的 block 作用域。例如，一个简单的 block 看起来像这样：

	^{
		// Block implementation here
	}

一个 block 也简单的是一个值，也有相关的类型。如同 int， float，或者 Objective-C 对象一样，一个 block 可以被赋给一个变量，然后就像其他变量一样使用。block 类型的愈发类似于函数指针。下面一个简单 block 的简单例子，它没有参数也没有返回值：

	void (^someBlock)() = ^{		// Block implementation here	};

这个 block 定义了一个名为 someBlock 的变量。这可能看起来有些奇怪，因为变量名在中间偏右，但是一旦你理解了这个语法，它就很容易阅读了。block 类型的语法结构如下：
	return_type (^block_name)(parameters)

为了定义一个返回 int 并带有个两个 int 参数的 block，你将会用到如下语法：

	int (^addBlock)(int a, int b) = ^(int a, int b){ 
		return a + b;	};

一个 block 可以被如同函数般使用。所以，例如，addBlock 可以被这么用：
	
	int add = addBlock(2, 5); //< add = 7

block 强大的功能是它允许你使用它申明处的上下文 。这以为这任何在 block 申明处的上下文中可用的变量在 block 中都是可用的。例如，你可以这样定义一个使用其他变量 block：

	int additional = 5;	int (^addBlock)(int a, int b) = ^(int a, int b){		return a + b + additional; 	};	int add = addBlock(2, 5); //< add = 12

缺省的，任何被 block 使用的变量都不能被 block 修改。在这个例子中，如果名为 additional 的变量在 block 中被改变了，编译器将会报错。可是，变量可以通过 __block 标识符申明为可被 block 改变。例如，一个 block 可以被用于遍历数组（见条目 48）以判断数组中有多少个数小于 2:

	NSArray *array = @[@0, @1, @2, @3, @4, @5]; 
	__block NSInteger count = 0;	[array enumerateObjectsUsingBlock:	^(NSNumber *number, NSUInteger idx, BOOL *stop){ 		if ([number compare:@2] == 	NSOrderedAscending) {			count++; 		}	}];	// count = 2

这个例子也展示了一个内联 block 的使用。传入 enumerateObjectsUsingBlock: 方法的 block 没有被赋给某个本地变量，而是为做了一个方法调用的内联声明。这个普遍的使用的 block 代码模式展示了为何它如此有用。在 block 成为语言的一部分之前，上面的代码需要通过传入函数指针或者一个遍历方法可以调用的选择器名来完成。状态必须手动传入传出，通常通过一个不透明的空指针，因此便引入了额外的代码，并将方法分割。申明一个内联 block 意味着业务逻辑都集中在一处。

当一个对象类型的变量被 block 使用时，一个 block 隐式地保留了它。它会在 block 释放时被释放。理解 block 这一点非常重要。block 本身可以被视为对象。实际上，blocks 可以对很多 Objective-C 对象能够响应的选择器做出反应。理解 block 被如同其他对象一样做了引用计数是很重要的。当 block 的最后的引用被删除了，它就被销毁了。这么做，任何被 block 使用的对象释放和保留都会平衡。

如果 block 在 Objective-C 类的实例方法中被定义，self 变量和类的实例在 block 都是可用的。实例变量总是可以可写的，无需显式地被申明为 __block。但是如果一个实例变量被读或写了， self 变量就隐式的被使用了，因为实例变量关系到实例。例如，考虑下面的名为 EOCClass 的方法中 block：

	@interface EOCClass
	- (void)anInstanceMethod { // ...		void (^someBlock)() = ^{			_anInstanceVariable = @"Something"; 			NSLog(@"_anInstanceVariable = %@",  _anInstanceVariable);		};		// ...	} 
	@end

EOCClass 的实例有一个 anInstanceMethod 方法，它如同 self 变量一样。很容易忘记 self 也被这种类型的 block 使用，因为它没有在代码中显式使用。可是，访问一个实例变量等效如下代码：

	self->_anInstanceVariable = @"Something";

这就是为什么 self 变量也被使用了。更多地，属性（见条目 6）会被用于访问实例变量，在这种例子中，self 是显式地：

	self.aProperty = @"Something";
	
然而，记住 self 是一个对象，在被 block 使用时会被保留是很重要的。这种情况 retain 循环，这常由 block 也被 self 指向的对象保留所引起。见条目 40 以获得更多信息。

#### Block 内部

Objective-C 中的每一个对象都占据一定的内存区。这块内存区因对象不同而有不同的大小，这取决于实例变量的数量和其包含的相关数据的多少。一个 block 本身也是一个对象，因为 block 内存区的第一个变量被定义为一个指向 Class Object 的指针，称为 isa 指针（见条目 41）。Block 占据的剩余内存包含了使其正常运行的各类信息。图 6.1 展示了 block 结构图的细节。

![alt Block](/images/blog/EffectiveObjC/6-1.jpeg "Block layout")

**图 6.1** Block 的内存结构

在结构图中最重要的东西是名为 invoke 的变量，它是一个指向 block 的实现的函数指针。函数原型要求至少一个 void * 类型的参数，它其实就是 block 自己。调用 block 其实就是嗲用一个将状态通过 void * 不透明指针传入的函数指针。Block 将之前使用标准 C 语言功能完成的功能包装成一个简洁易用的接口。

descriptor 变量是一个指向每个 block 都拥有的结构体的指针，它声明了 block 对象的大小，以及 copy 和 dispose 这些便利方法的函数指针。这些便利方法在 block 被拷贝和被销毁时被调用，例如，执行任何保留和释放时。

最后，一个 block 包含它所使用的变量的全部拷贝。这些拷贝被存储在 descriptor 变量之后，并占据了所有使用到的变量那么大的空间。注意这不意味着对象本身被拷贝了，而是只是对象的指针而已。当 block 运行时，被使用的变量被从这块内存区域读出，这就是 block 为何需要被像参数一样传入 invoke 函数。

#### 全局，栈和堆 Block
当 blocks 被定义是 ，分配给它的内存空间是在栈上。这意味着 block 只在它定义的作用域有效。例如，下面的例子是危险的：

	void (^block)();	if ( /* some condition */ ) {		block = ^{ 			NSLog(@"Block A");		};	} else {		block = ^{ 			NSLog(@"Block B");		};	}	block();

这两个被定义在 if else 表达式中的 blocks 被分配在栈上。当编译器为每一个 block 分配栈内存空间时，编译器在内存被分配发生的作用域的结尾可以重写这块内存。所以每个 block 只在各自的 if 表达式中被保证有效。这段代码会没有错误地通过编译，但是在运行时可能运行正常也可能运行错误。如果不走到制成重写给定 block 的代码，代码将无误运行，但是如果，崩溃将必然发生。

为了解决这个问题，可通过向 blocks 发送 copy 消息而被复制。这么做，block 被从栈拷贝到堆。一旦这么做了，block 就可被用于它定义处作用域之外的地方。一旦，被拷贝到堆，block 就变为一个引用计数对象。后续的拷贝将不再真正执行拷贝而是增加 block 的引用计数。当一个堆 block 不再被引用，它需要被释放，或者如使用了 ARC，将会被自动释放，或如果手动管理引用计数，则通过显式调用 release 释放。当引用计数降到 0 时，堆 block 如其他对象被销毁。一个栈对象无需显式地被释放，因为这已经被栈内存本身的机制（变量申明时被压入栈，作用域结尾时被全部弹出栈）自动处理了：这也是为何例子中的代码很危险。
 
根据以上想法，你可以简单地对 block 调用 copy 方法以使其安全：

	void (^block)();	if ( /* some condition */ ) {		block = [^{ 			NSLog(@"Block A");		} copy]; 	}	else {		block = [^{ 			NSLog(@"Block B");		} copy]; 	}	block();
 
代码现在安全了。如果使用手动引用计数，block 仍然需要在完成后被释放。

全局 block 是除栈和堆 blocks 外的另一种类别。不需要任何状态的 blocks，如不需要任何封闭作用域中的变量，都无需状态即可运行。Blocks 使用的整个内存区域在编译器都是可知的；所以全局 blocks 是被分配在全局内存中，而不是每次他们被使用时都被在栈中再创建一次。无法对一个全局 block 做拷贝操作，全局 block 也不可被销毁。如此 blocks，实际上是单例的。。如下是一个全局 block：

	void (^block)() = ^{ 
		NSLog(@"This is a block");	}; 

执行这个 block 所需的所有信息在编译时就已知了；因此，它可以是一个全局 block。这纯粹是一个减少不必要工作的优化，如这个简单的 block 被像那些需要在被拷贝或销毁时做大量工作的更复杂的 block 一样对待的话，全局 block 下这些工作就都被减免了。
 
#### 要点回归
* Blocks 是 C，C++，和 Objective-C 的闭包。
* Blocks 可以带参数，也可以有返回值。
* Blocks 可以分配在栈，也可分配在堆，也可以是全局的。一个分配在栈的 block 可以被拷贝到堆，在堆中它可以如标准 Objective-C 对象一样被引用。

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

Blocks 很容易引入 retain 循环，如果他们不被仔细考虑的话。例如，如下的类提供了一个下载特定 URL 的接口。一个被称为 completion handler 回调 block，当下载器开始运行时它可以被赋值。Completioan handler 需要被存为实例变量，以便在 request-completion 方法被调用时可用。

	// EOCNetworkFetcher.h	#import <Foundation/Foundation.h>	typedef void(^EOCNetworkFetcherCompletionHandler)(NSData *data);	@interface EOCNetworkFetcher : NSObject 
	@property (nonatomic, strong, readonly) NSURL *url; 
	- (id)initWithURL:(NSURL*)url;	- (void)startWithCompletionHandler: (EOCNetworkFetcherCompletionHandler)completion; 
	@end	
	// EOCNetworkFetcher.m	#import "EOCNetworkFetcher.h"	@interface EOCNetworkFetcher ()		@property (nonatomic, strong, readwrite) NSURL *url;	@property (nonatomic, copy) EOCNetworkFetcherCompletionHandler completionHandler; 	@property (nonatomic, strong) NSData *downloadedData;		@end
		@implementation EOCNetworkFetcher
	- (id)initWithURL:(NSURL*)url { 		if ((self = [super init])) {			_url = url; 		}		return self; 	}
	- (void)startWithCompletionHandler: (EOCNetworkFetcherCompletionHandler)completion	{		self.completionHandler = completion;		// Start the request		// Request sets downloadedData property		// When request is finished, p_requestCompleted is calle	}	- (void)p_requestCompleted { 		if (_completionHandler) {			_completionHandler(_downloadedData); 		}	}
	@end

另一个类可能创建一个网络下载对象并用它下载某个 URL 的数据，如此：

	@interface EOCClass (){ 
		EOCNetworkFetcher *_networkFetcher; 
		NSData *_fetchedData;	}	
	@implementation EOCClass
	- (void)downloadData {		NSURL *url = [[NSURL alloc] initWithString: @"http://www.example.com/something.dat"]; 		_networkFetcher = [[EOCNetworkFetcher alloc] initWithURL:url];		[_networkFetcher startWithCompletionHandler:^(NSData *data){ 			NSLog(@"Request URL %@ finished", _networkFetcher.url);			_fetchedData = data; 		}];	}
	@end

这段代码看起来很平常。但是你可能没意识到 retain 循环已经出现了。它源于 completion-handler block 引用了 self 变量，因为它给 _fetchedData 实例变量赋值了（见 条目 37 更多了解使用变量）。这意味着创建了 network fetcher 的 EOCClass 实例被 block 保留了。这个 block 被 network fetcher 保留了，network fetcher 被同一个 EOCClass 对象保留，因为它持有了一个强实例变量 network fetcher。图 6.2 展示了这个 retain 循环。

![alt Block Retain Cycle](/images/blog/EffectiveObjC/6-2.jpeg "Block Retain Cycle")
**图6.2 network fetcher 和 持有它的类之间的 retian 循环**

这个 retain 循环可以通过打破 _networkFetcher 实例变量的引用或 completionHandler 属性的持有来修正。这个破坏需要在这个 network fetcher 的 completion handler 完成的情况下进行，所以 network fetcher 直到它完成为止都是可用的。例如，completion-handler 可以变为如下：

	[_networkFetcher startWithCompletionHandler:^(NSData *data){ 
		NSLog(@"Request for URL %@ finished", _networkFetcher.url);
		_fetchedData = data;		_networkFetcher = nil;	}

retain 循环是使用 completion 回调 block 的 API 的一个通常问题，因此理解它很重要。通常，问题可以通过在合适的时机清除一个引用来解决；可是，它不能保证那个时刻总是会发生。在例子中，retain 循环只在 completion handler 运行时才被打破。如果 completion handler 永远不被运行，retain 循环将永远不会被打破，内存泄露就发生了。

另外一个潜在的 retain 循环通过 completion-handler block 引入。在 completion-handler block 引用的对象不再拥有它时，这个 retain 循环便发生了。例如，拓展前面的例子，不是消费者在 network fetcher 运行时，保持一个 network fetcher 的引用，而是有一个机制使它自己存活。当 network fetcher 开始时，它可能被加入到一个全局集合，例如一个 set。结束时则被剔除出集合。代码变化如下：

	- (void)downloadData {		NSURL *url = [[NSURL alloc] initWithString:@"http://www.example.com/something.dat"]; 		EOCNetworkFetcher *networkFetcher = [[EOCNetworkFetcher alloc] initWithURL:url]; 		[networkFetcher startWithCompletionHandler:^(NSData *data){ 			NSLog(@"Request URL %@ finished", 	networkFetcher.url);			_fetchedData = data; 		}];}

大多数网络库使用这种方法，因为它可以避免自己持有一个可用的 fetcher 对象。一个例子是来自 Twitter 库的 TWReqeust 对象。可是，EOCNetworkFetcher 的代码中，retain 循环仍然存在。它比之前的更为微妙，虽然源于 completion－handler block 引用了请求本身。这个 block 因此保留了 fetcher，fetcher 通过 completionHandler 保留了 block。幸运的是，修正是很简单的。记住 completion handler 被保存在属性中仅仅是因为它在后面会被用到。问题是一旦 completion handler 被运行了，它就不再需要持有 block 了。所以，简单的修正是改变如下方法：
 
 	- (void)p_requestCompleted {
 		 if (_completionHandler) {		_completionHandler(_downloadedData); 		}		self.completionHandler = nil;  	}

Retain 循环在请求完成时被解除了，而且 fetcher 对象将被销毁。这是要把 completion handler 传入 start 方法的好理由。如果，不将 completion handler 暴露为一个公共属性，你就不能在请求完成时清除它，因为那会打破你给予消费者的 completion handler 是公开的封装语义。在这种情况下，明智地打破 retain 循环唯一方法是强制消费者在自己的 handler 中清除 completionHandler 属性。但其实这并不明智，因为你不能假设消费者会这么，而不因此责备你造成了内存泄漏。

两个场景。他们是在使用 block 过程中容易犯的 bug；类似地，如果你小心一些，也很容易减少这类 bug。要点是考虑 block 会使用对象，而且保留它。如果任何这些问题都是由对象直接或见解地保留了 block，你可能需要考虑如何在正确的时刻打破 retain 循环。

#### 要点回归
* 注意 blocks 使用了直接或间接 retain 了 block 内的对象，会引入 retian 循环这样的潜在问题。
* 确保 retain 循环在恰当的时候被打破，绝不要将这个任务留给 API 的使用者们。

## 条目 41 ： 推荐使用 dispatch queues 来同步锁
有时在 Objective-C 中，你会遇到一些麻烦的代码，因为它会被多个线程访问。这种情况通常需要应用通过锁的使用而获得某种同步机制。在 GCD 之前，有两种方法完成同步，第一种是使用内建的同步块：

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
	
递归锁常通过 NSRecursiveLock 实现，它允许一个线程多次获得同一个锁而不会引发锁死。

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

这个模式背后的想法是所有对属性的访问操作都被同步了，因为取值器和赋值器所运行的 GCD 队列是同一个串行队列。撇开取值器中的 __block 关键字，使得 block 可以给变量赋值（见 条目 37），这个方法非常简洁。所有的锁都在 GCD 中被处理，而 GCD 是在底层实现的，做过了很多的优化。因此，不用担心线程方面的事情而能够专注与编写你的访问器代码。

可是，我可以更进一步。赋值器无需是同步的。给实例变量赋值的 block 无需返回任何东西。这意味着你可以将赋值器改为如下样子：

	- (void)setSomeString:(NSString*)someString { 
		dispatch_async(_syncQueue, ^{
			_someString = someString; 
		});
	}

这个简单的从同步分发改变到异步分发提供了一个在调用者看来更快的赋值器，但是读和写仍然被一个一个的串行执行。这是一个缺点，如果你做了检测，你可能会发现它有点慢；异步分发中，block 必须被拷贝。如果执行拷贝的时间和 block 的执行时间相当的话，它就会变慢。所以，在我们这个简单的例子中，它看起来是更慢了。可是，这个方法仍然是个好方法，如果将其理解为在 block 被分发执行很重的任务时的潜在选项的话。

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
由于 Objective-C 的极为动态的特性（条目 11），一些定义在 NSObject 中的方法允许你调用任何你想调用的方法。它们允许你推迟方法调用的执行，或者推迟决定它需要在哪个线程上运行。它们曾是十分有用的功能；可是现在，诸如 Grand Central Dispatch 和 block 这样的技术使它们变得不再重要。虽然你仍然可以看到使用它们的代码，但我鼓励你清除它们。

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
	
这个代码相当灵活，并常可以被用于简化复杂代码。另外一种用法是存储一个在某个事件发生后必须执行的选择器。无论哪种情况，编译器都直到运行时才知道哪个选择器会被执行。但是这个功能的代价是，如果你在 ARC 下编译，编译器会抛出如下警告：

	warning: performSelector may cause a leak because its selector is unknown [-Warc-performSelector-leaks]

你可能不希望这样！如果你确实不希望这样，你可能知道为什么你应该对这些方法小心。这个消息对你来说，看起来可能有些奇怪，你会想知道为何提到了泄漏。毕竟，你只是简单的试图调用一个方法。原因是编译器不知道哪个选择器将会被调用，因此不知道方法签名，返回类型，甚至不知道是否有返回值。编译器也不知道方法名，因此不能使用 ARC 内存管理规则决定哪些返回值应该被释放。由于这个原因，ARC 为了安全起见不会释放。可是，结果可能会造成泄漏，因为返回对象可能被保留过。

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

这个例子对前例做了细小的变化以展示出问题。在前两个选择器的情况下，ret 对象将需要被代码释放；而第三个选择器，则无需释放。不仅仅在 ARC 环境下，在严格遵守了方法命名指导的非 ARC 环境下，也会是如此。没有 ARC（因此没有编译器警告），ret 对象将需要被释放如果第一或者第二个条件为真时，但其它情况下则无需释放。这很容易被忽视，甚至静态分析器都不能帮助检查出随后的内存泄漏。这是一个慎重使用 performSelect 系列方法的理由。

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

所有这些限制都可以使用一个替代物解决。主要替代是使用 blocks（条目 37）。更进一步，blocks 配合 Grand Center Dispatch （GCD）使得你可以解决所有因线程关系而使用 performSelect 方法的情况。延迟执行可以通过 dispatch_after 完成，在另外一个线程执行可以通过 dispath_sync 和 dispath_async 完成。

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
* 系列方法有相当限制，要遵守返回值类型，传入方法的参数的数量也有限制。
* 允许在不同线程里执行的 performSelector 系列方法的地方，最好使用特定的 Grand Central Dispatch（GCD）调用 block 来代替。

## 条目 43 ： 知道何时使用 GCD 和 何时使用操作队列
GCD 是一项了不起的技术，但是有时使用标准库里的其它工具会更好。知道合适使用某种工具是重要的；使用错误的工具会导致代码难以维护。

GCD 的同步机制（见条目 41）强大得难有可与之匹敌的对手。同样的单一代码执行这个任务，可以通过 dispatch_once (见条目 45)实现。可是，使用 GCD 并不总是在后台执行任务的最佳方案。一个分离的但相关的技术，NSOperationQueue，允许你将操作（NSOperion 的子类）排入队列，操作会竞争运行。这与 GCD 的 dispatch queue 相似不是一个巧合。操作队列先于 GCD，但是无疑 GCD 是基于使得操作队列流行起来的同样的原则。实际上，从 iOS 4 和 Mac OS X 10.6 之后，操作队列就使用 GCD 实现。

第一个需要注意的不同是 GCD 是一个纯 C API，而操作队列是 Objective-C 对象。在 GCD 中，被排入队列中的任务是 block，这是一个很轻的数据结构（见条目 37）。另一方面，操作（Operation）则是 Objective-C 对象，因此它更重一些。不是说，GCD 不总是最佳选择？有时，这个消耗很小，但是使用对象的益处会远远超过负面因素。

通过使用 NSBlockOperation 或者 NSOperationQueue 的 addOperationWithBlock: 方法，操作队列的语法看起来和 GCD 非常相似。这是一些使用 NSOperation 和 NSOperationQueue 的好处：

* 取消操作

操作队列中，这很简单。运行时，执行 NSOperation 的 cancel 方法设置操作的内部标识，以告诉它不再运行，虽然它不能取消一个已经开始的操作。另一方面来说，GCD 队列没有办法取消一个已经启动的 block。机制是 “fire and forget”。可是在应用层面实现取消，还是有可能，但是要求编写大量代码，而这些代码已经以操作形式被编写过了。

* 操作依赖

一个操作如果它希望的话，可以依赖其他很多操作。这允许你创建一个操作层级，以指示某些操作只有在其他操作成功完成之后才能执行。例如，你可能有一个操作用于从服务器下载文件，这要求先下载一个 manifest 文件，其他文件的下载操作依赖于这个 manifest 文件。如果操作队列被设置为允许竞争执行，后续的下载可以并行执行，但是之后在所依赖的操作完成之后。

* 操作属性的键值观察（KVO）

有很多属性的操作很适合键值观察（KVO），例如 isCancelled 以判断它是否已经被取消了，以及 isFinished 以判断它是否已经完成了。如果你有代码需要知道特定的任务改变了状态，并给予任务比 GCD 更细粒度的控制的操作，就可以使用键值观察。

* 操作优先级

操作可以与优先级联系，以对队列中的操作排序。高优先级的操作将先于低优先级的操作被执行。排队算法是不透明的，但是大部分可以被观察出来。GCD 没有完成这类事情的直接方法。它有一个队列优先级，但是他们给整个队列设置优先级而不是单个 blocks。编写你自己的调度器并不是你真正希望做的事情。优先级因此是操作的一个很有用的功能。

操作也有一个关联的线程优先级，它决定当操作执行时，哪个优先级的线程将会执行。你可以在 GCD 中自己实现，但是操作将这变得更简单了，只需设置一个属性。

* 操作重用

除非你使用使用内建的 NSOperation 的具体子类，例如 NSBlockOperation，否则你必须创建自己的子类。这个累，作为一个普通的 Objective-C 对象，可以存储任何你想要的信息。当它运行时，它可以使用所有它存储的信息和使用所有该类定义的方法。这使得它比一个分发队列中的 block 要强大得多。顺应软件开发的 “Don‘t Repeat Yourself”（DRY）原则，这些操作类可以在你得代码里重用。

如你能看到的，这有很多好理由使用操作队列而不是分发队列。操作队列更多的是提供了一个你执行任务现成的解决方案。为了避免自己编写复杂的调度器，或者取消语法，优先级，你可以通过使用操作队列免费得到它们。

一个 API 利用了操作队列而不是分发队列的例子是 NSNotificationCenter，它有个方法你可以注册以观察一个提醒，收到提醒之后执行一个 block 而不是调用一个选择器。方法原型如下：

	- (id)addObserverForName:(NSString*)name 
					   object:(id)object
	 					queue:(NSOperationQueue*)queue 
	 			   usingBlock:(void(^)(NSNotification*))block
	 			   
代替操作队列，这个方法可以使用一个分发队列来处理提醒的毁掉 block。但是，这个设计决定是为了使用更高级别的 Objective-C API。这个案例中，两者在效率层面的差别很细微。这个决定更可能是因为使用分发队列会引入对于 GCD 的不必要的依赖；记住 blocks 不是 GCD，所以 block 本身不会引入这个依赖。或者也许开发者希望保持它们全部为 Objective-C。

你可能会常听到你应该尽可能使用高级别的 API，只在确实需要时才降级。我谨慎地认可这个口头禅。因为，可以使用高级别的 Objective－C 完成并不真正意味着它更好。两个都比较一下总是知道哪个是最好的最好方法。

#### 要点回归
* 分发队列不是任务管理和多线程的唯一解决方案。
* 操作队列提供了高级别的，Objective-C API，并能完成大部分 GCD 的工作。这些队列和 GCD 代码配合，也可以完成更为复杂的事情。

## 条目 44 ： 使用分发组以利用平台伸缩性

分发组（dispatch groups）是一个 GCD 功能，允许你很简单就能组合任务。你等待一个任务集合完成，或者通过任务完成时的回调被通知。基于几个理由，这个功能很有用，其中首要也是最有趣的是当你要竞争执行多个任务，但却需要知道何时它们全部完成。这有一个执行这样任务的例子，比如压缩文件。

一个分发组通过一下函数被创建：

	dispatch_group_t dispatch_group_create();

一个组是一个简单的数据机构，没有任何特别之处，不像分发队列，有一个标识。你有两种方法来关联任务和分发组。第一种是使用如下函数：

	void dispatch_group_async(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block);

这是普通 dispatch_async 函数的替代品，只是需要一个额外的参数 group，它标识要执行的 block 和哪个 group 关联。第二个将任务和分发组关联的方法是使用如下两个函数：

	void dispatch_group_enter(dispatch_group_t group); 
	void dispatch_group_leave(dispatch_group_t group);

前者导致。

因此，每次调用 dispatch_group_enter，这必须有一个对应的 dispatch_group_leave 调用。这类似于引用指针（见条目 29），这与 retains 和 releases 必须平衡以避免内存泄露一致。在分发组这个案例中，如果一个 enter 没有一个 leave 与之对应，那么 group 将永远不会结束。

下面的函数可以被用于等待一个分发组结束：

	long dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout); 

这使组等待并有一个超时时间。超时时间表示这个函数在等待组完成时应该阻塞多长时间。如果组在超时时间之前结束，返回 0；否则，返回非 0 值。常量 DISPATCH_TIME_FOREVER 可以被用于超时值，以指示函数应该永远等待，而永不超时。

下面函数是一个阻塞当前线程以等待分发组结束的替代方法：

	void dispatch_group_notify(dispatch_group_t group, 
								dispatch_queue_t queue,								dispatch_block_t block);

和 wait 函数细微的不同是，这个函数允许你指定一个会在组结束时运行在指定队列上的 block。如果当前线程不应该被阻塞，而你仍然需要知道何时所有的任务已经完成乐，那么这么做就是有用的。在 Mac OS X 和 iOS 中，例如，你应该不会阻塞主线程，因为主线程是执行 UI 绘制和事件处理的地方。

一个使用这个 GCD 功能的例子是执行一个数组中对象里的任务，并等待所有任务完成。下面代码做了这些：

	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	dispatch_group_t dispatchGroup = dispatch_group_create(); 
	for (id object in collection) {
		dispatch_group_async(dispatchGroup, queue, ^{ [object performTask]; });
	｝
	dispatch_group_wait(dispatchGroup,asad DISPATCH_TIME_FOREVER); 
	// Continue processing after completing tasks

如果当前线程不应该被阻塞，你可以使用 notify 函数替代等待：

	dispatch_queue_t notifyQueue = dispatch_get_main_queue(); 
	dispatch_group_notify(dispatchGroup,						   notifyQueue,					 	   ^{								// Continue processing after completing tasks							});

notify 回调应该被排入的队列完全取决于环境。这里，我已展示它是主队列，它是一个广泛使用。但是它也可以是任何自建的串型队列或者某个全局竞争队列。

在这个例子中，所有任务的分发的队列都是一样的。但，并不必须如此。你可能要将这些任务放在一个更高的优先级，但是仍然将它们组织到同一个分发组，并在它们全部完成时被通知。

	dispatch_queue_t lowPriorityQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0);
	dispatch_queue_t highPriorityQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0); 
	dispatch_group_t dispatchGroup = dispatch_group_create();	for (id object in lowPriorityObjects) { 		dispatch_group_async(dispatchGroup, lowPriorityQueue, ^{ [object performTask]; });	}
		for (id object in highPriorityObjects) { 		dispatch_group_async(dispatchGroup, highPriorityQueue, ^{ [object performTask]; });	}
	dispatch_queue_t notifyQueue = dispatch_get_main_queue(); 
	dispatch_group_notify(dispatchGroup, notifyQueue, ^{		// Continue processing after completing tasks	});

也许不是像上例中将任务提交到竞争队列，你可能使用分发组跟踪提交到串型队列的多任务。可是，如果所有任务被排入相同的串型队列，组并不是特别有用。因为任务将会串型执行，你可以简单地一个个排列任务，这等效于分发组地 notify 回调 block：

	dispatch_queue_t queue = dispatch_queue_create("com.effectiveobjectivec.queue", NULL);
	
	for (id object in collection) { 
		dispatch_async(queue, ^{ [object performTask]; });
	}
	
	dispatch_async(queue, ^{
		// Continue processing after completing tasks
	});

这段代码显示你不必总是使用分发组。有时，所需效果可以通过使用单个队列和标准异步分发来完成。

为什么我提到基于系统资源执行任务？好吧，如果你回顾分发到竞争队列的例子，这应该就清楚了。GCD 自动创建新线程或者重用旧的，使它看起来适合服务于队列中 block。在竞争队列的例子中，这可以是多线程，意味着多个 blocks 竞争执行。执行给定竞争队列的竞争线程数取决于系数，大部分基于系统资源，这都由 GCD 决定。如果 CPU 是多核的，一个有很多工作要做队列，将被多个线程执行。分发组提供了一个简单的方法来竞争执行一组给定的任务，并在这组任务完成时被通知。通过 GCD 的竞争队列的性质，任务将要被基于系统资源竞争执行。留给你的是编写你的业务逻辑，而不需要编写任何复杂的调度器去处理这些竞争任务。

循环一个 collection 并执行其中的元素中任务的例子可以通过使用另外一个 GCD 功能实现，如下：

	void dispatch_apply(size_t iterations, dispatch_queue_t queue, void(^block)(size_t));

函数执行 iterations 次 block，每次传入一个从 0 每次增加 1 的值。它像这样来使用：

	dispatch_queue_t queue = dispatch_queue_create("com.effectiveobjectivec.queue", NULL);

	dispatch_apply(10, queue, ^(size_t i){ // Perform task });

实际上，这等价于一个简单的从 0 迭代到 9 的 for 循环，如下：

	for (int i = 0; i < 10; i++) { 
		// Perform task	}

dispatch_apply 需要指出的关键点是队列可以是竞争队列。如果这样，block 将基于系统资源并行执行，就像分发组的例子一样。如果在 例子里的 collection 是一个数组，它可以用 dispatch_apply 重写如下：

	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);	dispatch_apply(array.count, queue, ^(size_t i){ 		id object = array[i];		[object performTask];	});

再说一次，这个例子表示分发组并不总是必须的。可是，dispatch_apply 阻塞着直到所有迭代都已完成。因为这个理由，如果你试图在当前队列上运行 blocks （或者当前队列之上的一个串型队列中），会导致锁死。如果你要让任务在后台线程执行，你就需要使用分发组。

#### 要点回归
* 分发组被用于组织一个任务集合。你可以选择在任务组执行完成时，接到通知。
* 分发组被用于在一个竞争的分发队列中执行多个任务。在这种情况下，GCD 基于系统资源，处理同一时期执行的多任务的排期。这些工作如果自己来写，将需要大量代码。

## 条目 45 ： 使用 dispatch_once 执行有线程安全单一执行要求的代码

单例模式－Objective-C 中并不陌生－常常被通过名为 sharedInstance 的类方法获取，它返回一个类的单例实例，而不是每次申请一个新的实例。一个通用的为名为 EOCClass 类的共享实例方法的实现如下：

	@implementation EOCClass
    	+ (id)sharedInstance {
       	 	static EOCClass *sharedInstance = nil; 
       	 	@synchronized(self) {
          		if (!sharedInstance) { 
               		sharedInstance = [[self alloc] init];
          		} 
	     	}
    	 	return sharedInstance;
        }
	@end

我发现单例生产是一个讨论热点，特别在 Objective-C 中。线程安全是争论的首要问题。先前的代码在同步块中创建了一个单例实例，以保证线程安全。无论好坏，这个模式被普遍采用，这类代码很常见。

可是，GCD 引入了一个使得创建单例实例更易实现的功能。这个函数如下：

	void dispatch_once(dispatch_once_t *token, dispatch_block_t block);

函数传入一个特别的 dispatch_once_t 类型，我们称其为 “token”，和一个 block。 函数确保给定的 token，block 被执行且仅仅被执行一次。block 总是执行一次，更重要的是，它是完全线程安全的。注意传入 token 需要确实相同，block 才会只执行一次。这意味着申明 token 变量为 static 或者申明为全局域。

用这个函数重写单例共享实例方法：

	+ (id)sharedInstance {
		static EOCClass *sharedInstance = nil; 
		static dispatch_once_t onceToken; 
		dispatch_once(&onceToken, ^{
			sharedInstance = [[self alloc] init]; 
		});
		return sharedInstance; 
	}
	 
使用 dispatch_once 简化了代码，并保证了整体的线程安全，所以你甚至不需要考虑锁和同步。所有这些都由 GCD 处理了。token 被申明为 static，是因为每次它需要是确实相同的。在 static 域中定义变量意味着编译器确保，每次只有一个单独的变量被重用，而不是每次创建新的变量。

更多的，dispatch_once 更为高效。它使用原子访问分发 token 以决定哪些代码已经被运行过了，而不是使用更重的每次都要求锁的同步机制。在我的 64 位， Mac OS X 10.8.2 上一个简单的参照，使用 @synchronized 方法访问 sharedInstance 方法，需要 dispatch_once 方法两倍的时间。

#### 要点回归
* 线程安全的单一的代码执行是一个普遍任务。GCD 通过 dispatch_once 提供了一个简单易用的工具。
* Token 应该被申明为 static 或者 global 这样使得传入每个需要单一执行的 block 的 token 都真正同一。

## 条目 46 ： 避免 dispatch_get_current_queue

#### 要点回归
* dispatch_get_current_queue 方法通常并不会按照你期望的那样执行。它已经被废弃了，现在只应该用于调试。
* 分发队列被按层级结构组织；因此，当前队列不能被简单地通过单个队列对象描述。
* 队列特定数据可以用于解决使用 dispatch_get_current_queue 的问题，它能避免由于非重入代码的死锁。
 