---
layout:    post
title:     Effective Objective-C 2.0 Chapter 7
category:  blog
description: 系统库
tags: ObjC
---

# 第 7 章 系统库
虽然在没有系统库的情况下使用 Objective-C 仍然是可能的，但是这么做是极为少见的。甚至标准根类，NSObject，也是 Foundation 库的一部分，而不是语言的一部分。如果你不用 Foundation 库，你必须写自己的根类，和自己的集合类，事件循环，和其他有用的类。而且，没有系统库，你还不能使用 Objective-C 开发 Mac OS X 和 iOS 应用。它们经过多年才发展成今天这样功能强大。因此，你可能会发现它们某些部分有些古老，用起来有些别扭，但是你也可能会发现瑰宝。


## 条目 47 ： 熟悉系统库
* 有很多系统库可用。其中最重要的是， Foundation 和 CoreFoundation，提供一个应用所需的大部分核心功能。
* 库为解决很多公共任务而存在，例如音频和视频处理过程，网络访问，和数据管理。
* 记住纯 C 编写的库对你来说如同 Objective-C 写的库一样重要。作为一个好的 Objective-C 开发者，你应该理解 C 的核心概念。

## 条目 48 ： 枚举块优于循环

### For loops
	
	NSArray *anArray = /*...*/
	for (int i=0; i<anArray.count; i++) {
		id object = anArray[i];
		// Do something with 'object'
	}

### Objective-C 1.0 使用 NSNumerator 的枚举
	
	NSArray *anArray = /*...*/
	NSNumerator *enumerator = [anArray objectEnumerator];
	id object;
	while ((object = [enumerator nextObject]) != nil) {
		// Do something with object
	}
	
### 快速枚举
	
	NSArray *anArray = /* ... */; 
	for (id object in anArray) {		// Do something with 'object'	}


	NSArray *anArray = /* ... */;	for (id object in [anArray reverseObjectEnumerator]) {		// Do something with 'object'	}

### 基于 block 的枚举

原型：

	- (void)enumerateObjectsUsingBlock: (void(^)(id object, NSUInteger idx, BOOL *stop))block

例子：

	NSArray *anArray = /* ... */;	[anArray enumerateObjectsUsingBlock:	^(id object, NSUInteger idx, BOOL *stop){ 		// Do something with 'object'		if (shouldStop) {			*stop = YES; 		}	}];


* 枚举集合可以通过 4 种方法完成。 for 循环是最基本的，其次是使用 NSEnumerator 的枚举和快速枚举。最现代和最先进的方法是使用 block-enumeration 方法。
* 基于 Block 枚举允许你无需额外代码，即可使用 GCD 竞争地执行枚举。这无法通过其他地枚举技术轻易完成。
* 基于 Block 的枚举，如果你知道精确的类型的话，改变 block 的签名以指明对象的精确类型。

## 条目 49 ： 在集合类的自定义内存管理中使用 Toll-free bridging

简单例子：

	NSArray *anNSArray = @[@1, @2, @3, @4, @5]; 
	CFArrayRef aCFArray = (__bridge CFArrayRef)anNSArray; 
	NSLog(@"Size of array = %li", CFArrayGetCount(aCFArray)); 
	// Output: Size of array = 5



* Toll-free bridging 允许你在 Foundation 库的 Objective-C 对象和 CoreFoundation 的 C 数据结构之间做强制转换。
* 使用 CoreFoundation 库创建集合允许你定义多种在集合处理其内容时调用的回调。通过 toll-free bridging 强制转换它们允许你得到一个有自定义内存管理语义的集合。

## 条目 50 ： 缓存中使用 NSCache 替代 NSDictionary

NSMutableDictionary 缺省会拷贝关键字，保留关键字对应的值。如果你要用不可以被拷贝的关键字会如何？其实，你不能在一个通常的 NSMutableDictionary 中使用他们，因为你遇到如下运行时错误：

	*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[EOCClass copyWithZone:]: unrecognized selector sent to instance 0x7fd069c080b0'

这个错误意味着这个类不支持 NSCopying 协议，因为 copyWithZone: 没有被实现。通过使用降入到 CoreFoundation 库级别创建字典，你可以改变内存管理语义，并创建一个保留关键字而不是拷贝关键字的字典。

* 如果 NSDictionary 被用作缓存，考虑使用 NSCache。缓存提供最佳的清除行为，线程安全，并且不象字典那样需要拷贝关键字。
* 使用计数极限和耗费极限构成定义对象应该从缓存清除的指标。但是不要依赖这些指标为强制限制；他们只是缓存的指导意见。
* 在缓存里使用 NSPurgeableData 对象，以提供清除数据时，也可以自动从缓存清除。
* 如果使用正确，缓存能使你的应用响应更快。仅仅缓存那些计算代价昂贵的数据，例如需要从网络或者硬盘读取的数据。

## 条目 51 ： 保持 initialize 和 load 实现的简洁

* 类进入 load 阶段会调用 load 方法，如果 load 方法被实现了的话。因为类比目录要先加载，所以这个方法也可以出现在目录里。不象其他方法，load 方法不易重写。
* 在类被第一次使用之前，它会收到 initialize 方法调用。这个方法可以重写，常用于检查哪个类正在被加载。
* load 和 initialize 的实现都应该保持简洁，这有助于保持应用的快速响应以及减少引入循环依赖的可能性。
* initialize 方法可以用于为不能再编译器使用的全局变量赋值。

## 条目 52 ： 记住 NSTimer 会持有它的目标

扩展 NSTimer 通过 block 回调：

	#import <Foundation/Foundation.h>	@interface NSTimer (EOCBlocksSupport)	+ (NSTimer*)eoc_scheduledTimerWithTimeInterval:(NSTimeInterval)interval											  block:(void(^)())block 											repeats:(BOOL)repeats;	@end

	@implementation NSTimer (EOCBlocksSupport)	+ (NSTimer*)eoc_scheduledTimerWithTimeInterval:(NSTimeInterval)interval											  block:(void(^)())block repeats:(BOOL)repeats	{		return [self scheduledTimerWithTimeInterval:interval											  target:self										    selector:@selector(eoc_blockInvoke:)								            userInfo:[block copy] 								            repeats:repeats];	}	+ (void)eoc_blockInvoke:(NSTimer*)timer { 
		void (^block)() = timer.userInfo;		if (block) {			block(); 		}	}
	@end

使用方法：

	- (void)startPolling {
		_pollTimer = [NSTimer eoc_scheduleTimerWithTimeInterval:5.0
												block:^{
												[self p_doPoll];
												}												repeats:YES];
	}

* 一个 NSTimer 对象保留它的 target，直至计时器因启动后无效或者显式调用 invalidate 而无效。
* 保留循环（retian cycles）很容易通过使用重复的计时器或者因为计时器的 target 保留了计时器而引入。
* NSTimer 的一个使用 blocks 的扩展可以被用于打破保留循环（retain cycle）。直到这个成为 NSTimer 的公共接口，这个功能都需要通过目录来添加。
 
