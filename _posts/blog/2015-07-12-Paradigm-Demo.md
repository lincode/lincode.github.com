---
layout: post
title: 浅析编程范式－以 Swift 为例
description: 本文主要基于一个小例子的演化过程，简单描述和比较了一下背后涉及的几种编程范式。编程的主要的工作是在对现实世界的各种事物和现象进行抽象。不同的编程范式会有不同的抽象方法，和适用领域。在使用Swift这种支持多种编程范式的编程语言时，理解多种编程范式将对更好的使用这门语言有很大帮助。
category: blog
---

文章会描述一个简单的小例子，例子中会基于几种编程范式做一系列演化。伴随这些演化，会简单描述和比较涉及到的几种编程范式。

例子中使用的编程语言是Swift。Swift是一门典型的多范式编程语言，使用Swift可以写出经典的面向对象代码；如果程序员具有函数式思维，使用Swift也可以写出函数式风格的代码；由于Swift对泛型的良好支持，任何时候程序员都可以使用范型提高代码的抽象层次。这使得可以使用Swift这一门语言来讲解多种编程范式。

## 过程式编程的循环

例子很简单，我们将从一个数组开始。现在需要将给定的数组的每个元素加一，以获得一个新数组。

在如Java这样的较纯粹的面向对象编程语言中，我们可能有类似这样的写法：

	func incrementArray(xs: [Int]) -> [Int] {
  		var result: [Int] = []
  		for x in xs {
    		result.append(x + 1)
  		}
  		return result
	}

当然我们现在使用的是Swift，Java的写法也不会有太大区别。例程中，我们编写了一个函数，函数要求一个整型数组作为传入参数，并返回一个整型数组作为返回值。在函数中，我们声明了一个新数组，然后循环遍历传入的数组，把数组的每个元素取出加一，再存入新数组，最后返回新数组。这里其实没有涉及到类和对象的概念，主要还是过程式编程的概念：循环遍历。但面向对象编程继承了几乎过程式编程的全部概念。现在，由于面向对象编程的普及，更多的时候我们是在面向对象编程看到这样风格的代码。其实，用C语言这样过程式编程语言编写以上代码，形式几乎是一样的。

## 函数式编程的递归

与函数式编程处理遍历的方法做个比较可能更清楚一些。在函数式编程语言中，没有循环遍历这种概念，通常的遍历数组的方法会是递归。在函数体内，会先处理数组的第一个元素，再将数组除第一个元素外的其他元素组成的新数组传人函数本身，再次调用。追随函数的调用轨迹，你会发现函数多次调用自己本身，直到触发边界条件，这就是递归。如果使用Scheme编写这个函数，会得到类似如下的代码：

	(define increment
	  (lambda (lat)
    	(cond
      	((null? lat) (quote()))
      	(else (cons (+ (car lat) 1) (increment (cdr lat)))))))

我们看不到`for`这类代表循环的关键字，而是在increment函数体看到它调用了自己：`(increment (cdr lat))`；对数组元素的处理则是：`(+ (car lat) 1)`，这是一个前缀操作，用通常的中缀表示是`(car lat) + 1`；边界条件是`((null? lat) (quote()))`，在传入空数组时，返回空数组终止了递归调用。其中，`car`是取数组第一个元素，`cdr`是取除第一个元素外的剩余元素，`cons`是连接两个参数成为一个数组。

循环和递归在这个例子中并没有表现出太大差别，只是处理数据的不同方法。

## 函数式编程的函数

这个函数到现在为止都很好地完成了任务。但如果，我们又有一个新的需求：给一个数组，将数组的每个元素乘以二，以获得一个新数组。和上面的例子差别不大，我们可以很容易地写出如下类似代码：

	func doubleArray(xs: [Int]) -> [Int] {
  		var result: [Int] = []
	  	for x in xs {
    		result.append(x * 2)
  		}
  		return result
	}

这个新函数也能很好地完成任务。但是，我们应该能发现这两个函数是如此的类似，仅有的差别是如何处理旧数组的每个元素，是`x + 1`还是`x * 2`。

写代码一条简单的原则是避免重复。遇到这种相似的代码的时候，我们就应该想一下如何避免这种重复呢？这就是一种抽象过程，把相似的东西提炼出来，从而可以在不同的地方反复使用。

这个例子中，很明显，我们应该把重复的循环部分抽象出来。为此，我们编写了如下函数：

	// Abstract the loop
	func computeIntArray(xs: [Int], f: Int -> Int) -> [Int] {
  		var result: [Int] = []
  		for x in xs {
    		result.append(f(x))
  		}
  		return result
	}

这个函数与前两个函数相比较的区别是多了一个传入参数，更重要的是这个参数是一个函数。我们把对旧数组每个元素的处理过程，以函数的形式传入。这样，就能灵活应对对数组元素的各种处理，而不用为每种处理过程都编写一个对应函数了。这里我们使用到了一个函数式编程的概念：高阶函数。可以将其他函数作为参数或者返回结果的函数被称为高阶函数。

函数式编程是一种以数学函数为程序语言建模的核心的编程范式。就像面向对象语言的主要抽象方法是类，函数式编程语言中的主要抽象方法是函数。在函数式编程中，函数是核心概念，是“头等公民”，函数被赋予了更多职责，拥有更多灵活的使用方式。

可以与面向对象编程语言比较一下，Java在支持闭包之前，算得上一门较纯粹的面向对象编程语言。在这样的纯粹的面向对象语言中，函数的传入参数只能是对象或者Int这样基本数据类型（primitive type）。在Java中，要把一个计算过程（函数）传入函数，只能先将函数放在一个类中，再把包裹了函数的对象作为参数传入函数。这个类似运载工具的对象通常会是一个匿名类，因为它别无它用，存在的目的仅仅是为了满足Java一切都是对象的哲学。在Java中，我们常见的Listener大部分都只是为了装载回调操作。

函数式编程给予了函数更“正常的”地位，使得函数可以有更灵活的用法。这个例子中，我们使用函数式编程范式，使得抽象出循环和循环里的计算过程成为可能。

这样，我们就可以更优雅地写出这两个函数：

	func incrementArray2(xs: [Int]) -> [Int] {
  		return computeIntArray(xs) { x in x + 1 }
	}

	func doubleArray2(xs: [Int]) -> [Int] {
  		return computeIntArray(xs) { x in x * 2 }
	}

## 泛型编程的类型

接下来，我们又有了第三个函数：要求判断给定整数数组每个元素的是否是偶数，并将结果存在一个新数组返回。这时我们发现上面的computeIntArray并不能完成这个任务。因为，computeIntArray的返回数组声明为`[Int]`，而新需求中的返回数组存储的却是布尔类型，类型声明是`[Bool]`。

我们需要这样改写这个函数：

	func computeIntArray2(xs: [Int], f: Int -> Bool) -> [Bool] {
  		var result: [Int] = []
  		for x in xs {
    		result.append(f(x))
  		}
  		return result
	}

我们自然会发现computeIntArray2和computeIntArray区别是如此微小，以至于我们不由自主再次地想把共同点抽象出来，以在不同的地方重复使用。我们发现这两个函数的区别主要在数据类型上。如果，我们可以把数据类型作为参数声明，在使用时再指定具体类型，这个抽象其实就完成了。这其实就是泛型。泛型为遍程语言提供了更高层级的抽象，即参数化类型。

有了泛型编程的概念，我们应该会这么重写这个函数：

	// Abstract the type with generic
	func genericComputeArray<U>(xs: [Int], f: Int -> U) -> [U] {
  		var result: [U] = []
	  	for x in xs {
    		result.append(f(x))
  		}
  		return result
	}

这样，我们就可以使用新的泛型函数，为新需求编写如下函数实现：

	func isEvenArray(xs: [Int]) -> [Bool] {
  		return genericComputeArray(xs, { (x: Int) -> Bool in
    		return x % 2 == 0
  		})
	}

由于Swift有着一个不错的类型推断系统，以及有在函数最后一个参数是闭包时可以将闭包写在括号外面的语法糖。我们可以把这个函数写得更美观紧凑一些：

	func isEvenArray(xs: [Int]) -> [Bool] {
 		return genericComputeArray(xs, { (x: Int) in x % 2 == 0 })
	}

更进一步，我们还可以观察到genericComputeArray只参数化了返回数组的类型，那么传入数组的类型是否也可以参数化呢？当然是可以的，这样我们就会得到一个更通用的对数组的处理方法，对数组元素的类型没有要求，对返回数组元素的类型也没有要求。这个函数可能会是这个样子：

	func map<T, U>(xs: [T], f: T -> U) -> [U] {
  		var result: [U] = []
  		for x in xs {
    		result.append(f(x))
  		}
  		return result
	}

我们将函数的名字改成了：`map`。这是因为Swift其实已经存在这样的函数，它的名字就是`map`。只是在Swift中，`map`是数组的一个成员方法。利用Swift已经提供的`map`函数，我们就可以简单而优雅地完成上面提到的三个函数：

	// Final in Swift
	func incrementArray3(xs: [Int]) -> [Int] {
	  return xs.map { x in x + 1 }
	}

	func doubaleArray3(xs: [Int]) -> [Int] {
  		return xs.map { x in x * 2 }
	}

	func isEvenArray3(xs: [Int]) -> [Bool] {
  		return xs.map { x in  x % 2 == 0 }
	}

也许会有朋友对如何在函数式编程语言中如何实现和使用`map`感兴趣。这里给出了一个简单的Scheme语言实现，同样没有循环，使用递归实现：

	(define map
  		(lambda (fn lst)
    		(cond ((null? lst) (quote ()))
          		(else (cons (fn (car lst)) 
                	      	(map fn (cdr lst)))))))

	(define (increment lst)
  		(map (lambda (x) (+ 1 x)) lst))

	(define (double lst)
  		(map (lambda (x) (* 2 x)) lst))

	(define (iseven lst)
  		(map (lambda (x) (even? x)) lst))

从这个例子中可以看到，一个简单的`map`函数背后其实有多种编程范式可以探究。理解这些编程范式，将有助于你使用Swift这类多范式编程语言。

## 总结

本文主要基于一个小例子的演化过程，简单描述和比较了一下背后涉及的几种编程范式。编程的主要的工作是在对现实世界的各种事物和现象进行抽象。不同的编程范式会有不同的抽象方法，和适用领域。在使用Swift这种支持多种编程范式的编程语言时，理解多种编程范式将对更好的使用这门语言有很大帮助。

如果，你对Swift的编程范式感兴趣。可以试着阅读另外一篇文章[《多范式编程语言－以 Swift 为例》](http://lincode.github.io/Swift-Paradigm/)，这篇文章中有更详细的描述。