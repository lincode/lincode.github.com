---
layout:     post
title:      Swift 的惰性计算
category: blog
description: Swift 支持函数式编程，这一篇介绍Swift的更高层级的抽象
tags: blog
---

Swift支持函数式编程，函数式编程提供更高层级的抽象。

### 更高层级的抽象

现代编程语言的发展有一个显著的趋势，就是提供更高层级的抽象，语言必须负担更多的责任。这让程序员远离繁琐的细节，比如，内存分配，垃圾回收，指针，对系统底层操作等。Java程序员先对于C和C++程序员更为轻松的一点是，Java程序员可以让语言来解决所有内存问题，从而可以更加专注于自己的核心业务。更高层级的抽象让程序员更多地关注在代码能做什么，而不是怎么做上。随着时间的推移，编程语言的这种发展趋势会更明显，因为这意味着更高的生产力。

而函数式编程对这一思想的贡献主要表现在，对我们编程中经常用到遍历也提供了更高层级的抽象。程序员在使用更高层级的抽象同时需要放弃对细节的控制。但是，这并不意味着无法在需要的时候回收控制。以函数式思维的一个重要方面是知道放弃多少控制，以及何时放弃。

### filter

一个简单的例子，找出1到10这个数组里的奇数。
	
	var odds = [Int]()
	for i in 1...10 {
  		if i % 2 == 1 {
    		odds.append(i)
  		}
	}

	println(odds)

输出结果为：[1, 3, 5, 7, 9]。而函数式的写法为：

	odds = Array(1...10).filter { $0 % 2 == 1 }
	println(odds)

输出结果当然是相同的：[1, 3, 5, 7, 9]。函数式的写法更简短一些，但更重要的区别是它放弃了对循环的控制，我们不知道filter的实现细节。因为，在这个场景中，我们除了需要知道filter要求我们提供一个用于依次处理数组中元素的闭包外，就无需知道其他细节了。

也许会有人关心filter是如何实现的，我们可以试着自己实现一个`myFilter`函数：

	extension Array {

		func myFilter<T>(resource:(T) -> Bool) -> [T] {
    		var results = [T]()
    		for i in self {
      			let item = i as T
      			if resource(item) {
        			results.append(item)
      			}
    		}
    		return results
  		}
  	}

这只是模拟了Swift里filter的实现。语言内部的实现并不一定是完全如此，Swift的编写人员会更多的优化空间。比如，未来在在多核机器上，也许可以对数组各元素并行计算。当然，获得这些收益的前提是放弃对遍历的控制。

### reduce

再来一个稍微复杂一些的例子，计算找出1到10这个数组里的奇数的和。使用Java语言的思维，通常的写法会是这样：

	var odds = [Int]()
	for i in 1...10 {
  		if i % 2 == 1 {
    		odds.append(i)
  		}
	}

	var sumOdds = 0
	for i in odds {
  		sumOdds += i
	}

	println(sumOdds)

结果是25。而函数式版本会是这样：

	let sum = Array(1...10)
			.myFilter { (i) in i % 2 == 1}
			.reduce(0) { (total, number) in total + number }
	println(sum)

下面是一个模拟Swift的reduce函数的实现：

	extension Array {

  		func myReduce<T, U>(seed: U, combiner:(U, T)-> U) -> U {
    		var current = seed
    		for item in self {
      			current = combiner(current, item as T)
	    	}
	    	return current
  		}
	}

模拟reduce的实现只是为了做练习，以及更好的理解它们是如何运行的。Swift的真实实现也许并不像这样。函数式编程的思维鼓励程序员把更多的精力放在代码能做什么，而不是代码怎么做上。具体到这个例子上，也就是说，程序员应该不用关心reduce是如何实现的，而只需要知道如何使用它。


### map

Array 还有一些处理数组的函数，比如，map。map的声明如下:

    func map<U>(transform: (T) -> U) -> [U])

意味着，它会以传入的闭包处理数组的每个元素，然后返回一个同长的数组。

举个例子，如果你想把一串数字都加上货币单位：

	let moneys = Array(1...10).map { (num) -> String in "\(num)¥" }


### List Comprehension

值得一提的是Haskell, Scala, Python中都支持的List Comprehension，到现在（2015年2月）为止，Swift还不支持。也就是说，在Swift中，你还无法写出如下代码：

	var evens = [ x for x in 1...10 if x % 2 == 0]

这是一个小小的遗憾。但是在任何情况下，filter，map 和 reduce 都可以替代上述List Comprehension语法。而且代码看起来并没有更复杂。


### 总结

放弃了对遍历的控制有如下好处：

- 首先，代码更为精炼了；
- 然后，如果语言层面能出现对循环遍历的优化，我们就能自动获益。比如，现在能想到的一个优化是，让闭包并行处理数组里面的各个元素。
- 最后，更高的抽象带来了更容易理解的代码。语言的表达方式更贴近于人类的通常思维方式，而不是要适应机器的计算方式。

Swift当然不会逼迫你放弃对循环内部逻辑的控制。在你需要更细力度的控制遍历时，你仍然可以使用 for 语法来随心所欲的控制循环。但是，绝大部分情况下，你可能并不需要这么做。程序员应该放弃控制一切的想法，把可以交给语言完成的都交给语言完成。这是对生产力的重要解放。
