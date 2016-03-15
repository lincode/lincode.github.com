---
layout: post
title: Swift 函数式编程－惰性计算
category: blog
description: “Swift 的编程范式” 之三：Swift 支持函数式编程，这一篇介绍 Swift 中的惰性计算。
tags: blog
---

Swift支持函数式编程，这一篇介绍Swift的惰性计算。

### 惰性计算

惰性计算是函数式编程语言的一个特性。在使用惰性计算时，表达式不在它被绑定到变量之后就立即求值，而是在该值被取用的时候求值。惰性计算有如下优点：

- 首先，你可以用它们来创建无限序列这样一种数据类型。因为直到需要时才会计算值，这样就可以使用惰性集合模拟无限序列。

- 第二，减少了存储空间。因为在真正需要时才会发生计算。所以，节约了不必要的存储空间。

- 第三，减少计算量，产生更高效的代码。因为在真正需要时才会发生计算。例如，寻找数组中第一个符合某个条件的值。

在纯函数式编程语言，如Haskell中是默认进行惰性求值的。所以，Haskell被称为惰性语言。而与之相对的大多数编程语言如Java，C++ 的求值都是严格的，或者说是及早求值。Swift默认是严格求值的，也就是每一个表达式都需要求值，而不论这个表达式在实际中是否确实需要求值。但是，Swift作为支持多种范型的编程语言，也同时提供语法来支持惰性求值。

### 内建 lazy 函数

Swift中，如果需要惰性计算，就要显式地将一个序列转化为惰性序列。转化方法是使用Swift内建的`lazy`函数。它有四个重载实现。编译器会为你选择最正确的实现。

如果传入lazy的是Sequence（实现了SequenceType协议的类或者结构体），返回的会是`LazySequence`；如果传入一个Collection（实现了CollectionType协议的的类或者结构体），返回的会是`LazyForwardCollection`, `LazyBidirectionalCollection`, 或者`LazyRandomAccessCollection`。
下面是lazy函数的四个函数重载函数的函数原型：

	func lazy<S: SequenceType>(s: S) -> LazySequence<S>
	func lazy<S: CollectionType where S.Index: ForwardIndexType>(s: S) -> LazyForwardCollection<S>
	func lazy<S: CollectionType where S.Index: BidirectionalIndexType>(s: S) -> LazyBidirectionalCollection<S>
	func lazy<S: CollectionType where S.Index: RandomAccessIndexType>(s: S) -> LazyRandomAccessCollection<S>

如果，传入一个Array，返回的将是`LazyRandomAccessCollection`类型。LazyRandomAccessCollection是惰性集合。下面展示了一个将Array变为惰性序列的例子：

	let r = 1...3
	let seq = lazy(r).map {
  		(i: Int) -> Int in
  		println("mapping \(i)")
  		return i * 2
	}

	for i in seq {
  		println(i)
	}

将获得如下结果：

	mapping 1
	2
	mapping 2
	4
	mapping 3
	6

这显示了seq是一个惰性序列。它的值只有在需要时才会真正发生计算。

### Generator

Swift中，Generator是任何实现了GeneratorType协议的类或者结构体。Generator可以理解为一个序列生成器。GeneratorType协议要求定义一个名为`Element`的别名，并实现一个`next`方法。

GeneratorType协议实现如下：

	protocol GeneratorType {
  	  	typealias Element
	  	mutating func next() -> Element?
	}

语句`typealias Element`要求实现这个协议的类必须定义一个名为Element的别名，这样一定程度上实现了泛型协议。协议同时要求实现`next`函数，其返回值是别名中定义的`Element`类型，next函数代表生成器要生成的下一个元素。

下面代码实现了一个菲波那契数列生成器：

	class FibonacciGenerator : GeneratorType {

		var current = 0, nextValue = 1  

	  	typealias Element = Int
  
		func next() -> Element? {
    		let ret = current
    		current = nextValue
    		nextValue = nextValue + ret
    		return ret
  		}
	}

下面代码打印出10个菲波那契数列，以显示如何使用生成器：

	var n = 10
	var generator = FibonacciGenerator()
	while n-- > 0 {
  		println(generator.next()!)
	}

Generator是Sequence和Collection的基础。

### Sequence

Sequence是任何实现了SequenceType协议的类或者结构体。Sequence可以理解为一个序列。SequenceType协议要求定义一个名为Generator，类型为GeneratorType的别名，并要求实现一个返回生成器Generator的函数。

SequenceType协议如下：

	protocol SequenceType : _Sequence_Type {
    	typealias Generator : GeneratorType
    	func generate() -> Generator
	}

类似于GeneratorType协议，`typealias Generator : GeneratorType`要求实现这个协议的类必须定义一个名为Generator类型为GeneratorType的别名。协议同时要求实现一个名为`generate`的函数，其返回值为别名`Generator`定义的类型，这个类型应该实现了上文提到的GeneratorType协议。也就是说Sequence其实是包含一个生成Generator的函数的类。

下面代码使用上文中提到的菲波那契数列生成器，实现了一个菲波那契数列：

	class FibonacciSequence: SequenceType
	{
  		typealias GeneratorType = FibonacciGenerator

		func generate() -> FibonacciGenerator {
    		return FibonacciGenerator()
		}
	}

下面代码打印了10个菲波那契数列，以显示如何使用该序列：

	let fib = FibonacciSequence().generate()
	for _ in 1..<10 {
  		println(fib.next()!)
	}

符合SequenceType的序列有可能成为惰性序列。成为惰性序列的方法是对其显示的调用lazy函数：

	let r = lazy(stride(from: 1, to: 8, by: 2))
	
函数stride返回一个结构体`StrideTo`，这个结构体是Sequence。所以，lazy函数返回一个`lazySequence`对象。

### Collection

Collection是实现了CollectionType协议的协议的类或者结构体。CollectionType协议继承了SequenceType协议。所以，Collection也都实现了SequenceType，它同时也是Sequence。

CollectionType协议如下：

	protocol _CollectionType : _SequenceType {
    	typealias Index : ForwardIndexType
    	var startIndex: Index { get }
    	var endIndex: Index { get }
    	typealias _Element
    	subscript (_i: Index) -> _Element { get }
	}
	
	protocol CollectionType : _CollectionType, SequenceType {
    	subscript (position: Self.Index) -> Self.Generator.Element { get }
    }
    
所以，CollectionType协议首先实现了SequenceType协议，并要求实现一个`subscript`方法以获取序列中每个位置的元素值。

Swift中，大量内置类如Dictionary，Array，Range，String都实现了CollectionType协议。所以，Swift大部分容器类都可以变为惰性序列。

	// a will be a LazyRandomAccessCollection
	// since arrays are random access
	let a = lazy([1,2,3,4])
 
	// s will be a LazyBidirectionalCollection
	let s = lazy("hello")


### 总结

Swift里的集合数据结构默认是严格求值的。但是，Swift也提供了惰性语法，在需要惰性时，你需要显式声明。这为开发者在Swift中使用惰性提供了条件。