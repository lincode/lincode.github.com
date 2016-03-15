---
layout: post
title: Swift 的泛型编程
category: blog
description: “Swift 的编程范式” 之五：Swift 支持泛型编程，简单总结一下 Swift 中的泛型。。
tags: blog
---

## 泛型

泛型编程允许定义参数化的类型，使得函数和类不再被限制在某种类型。泛型一般出现在强类型语言中，因为对于弱类型语言并无必要。强类型语言对类型有更严格类型检查，需要泛型机制在加强类型安全的同时，又对类型进行了抽象，从而减少了类型转换的次数。本质来说，泛型是对类型的抽象，它带来了更高层次的抽象，避免了重复的代码。

泛型实现中，比较有影响的是C++的STL（Standard Template Library）。STL以模板来定义泛型：把原本特定于某个类型的算法或类当中的类型信息抽象出来，编写成一些包含参数的类，参数可以代表类型，这样的参数化的类就是模板。Haskell中由于使用在函数参数中，泛型被称为参数多态。除此之外，Java，C#，Scala和Swift也都支持泛型。

下面几节主要讨论Swift中的泛型。

## 泛型函数

泛型函数通过将函数参数和返回值定义为泛型类型，使得函数可以作用于任何适合的类型。

一颗用于交换两个值的泛型函数，在这个函数中只标记了一个泛型占位类型符T，作为类型参数，而没有对这个类型参数进行其他约束。所以，该函数可以用于任何类型。唯一的要求是，用于交换的两个参数的类型必须一致。

	func swapTwoValues<T>(inout a: T, inout b: T) {
		let temporaryA = a
		a = b
		b = temporaryA
	}

可以定义多类型参数，同样都命名在尖括号中，以逗号分隔。类型参数可以被用于定义个函数的参数类型，也可以用于定义函数返回值类型，或用于定义函数体中的类型。

## 泛型类型

除了泛型函数之外，Swift 还允许自定义泛型类型，这样的自定义泛型类，结构体和枚举就可以作用于任何类型，和Swift已经提供的Array和Dictionary的用法相同。

### 泛型结构体，枚举和类

用一个栈（Stack）的例子展示泛型结构体的定义方法，泛型枚举和泛型类的定义和使用方法是相同的。

	struct Stack<T> {
  		var items = [T]()

  		mutating func push(item: T) {
    		items.append(item)
  		}

	 	mutating func pop() -> T {
    		return items.removeLast()
  		}
	}

泛型类型参数T被用在了三个地方：

- 创建数组items时，指定了了items中可以存储的数据类型；
- 指定了函数push的参数类型；
- 指定了函数pop的返回值类型；

### 泛型协议

其实Swift中没有提供类似结构体或类那样的方法来定义泛型协议。但我们可以使用`typealias`关键字定义一个该协议的关联类型，这样一定程度上可以模拟泛型的效果。

举一个例子，Generator是任何实现了GeneratorType协议的类或者结构体。可以将Generator理解为一个序列生成器。

	protocol GeneratorType {
		typealias Element
		mutating func next() -> Element?
	}

GeneratorType协议对实现该协议的类有两点要求：

- 定义一个别名为`Element`的关联类型;
- 实现一个`next`方法。

协议要求实现方法很常见。这里的重点在语句`typealias Element`。它要求实现这个协议的类必须定义一个别名为Element的关联类型。协议并没有限制关联类型Element指代何种类型，而由实现该协议的类指定Element指代的具体类型。这和泛型的概念异曲同工，一定程度上实现了泛型协议。

下面给出一个现实这个协议的例子，它是一个菲波那契数列生成器：

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

这个例子中，别名Element指代的类型是Int。

### 泛型扩展(Extension)

对于泛型类型，我们可以使用 extension 关键字为泛型类型扩展新方法。

如果需要对上面提到的代表栈的结构体`Stack`进行扩展，添加一个方法top，获取栈顶值，但不将其弹出：
	
	extension Stack {
		func top() -> T? {
    		return items.last!
  		}	
	}

扩展中，可以直接使用Stack中已经定义的泛型参数T。

在扩展中使用泛型需要注意几点：

- 在扩展中，不能在修改已经定义的泛型参数，也就是说不能添加新的约束；
- 在扩展中，不能添加整个类型可用的新的泛型参数；
- 泛型类型已经定义的泛型符号，我们可以在扩展中直接使用;
- 在扩展中，新定义的方法中可以定义新的泛型参数，只要名字不和已经定义的泛型参数冲突;

## 类型约束

之前所举的例子中的泛型函数和泛型结构体，都没有对泛型参数做约束。也就是说，函数`func swapTwoValues<T>(inout a: T, inout b: T)`和结构体`Stack<T>`中的，泛型参数T可以被用于代表任何类型。但是，编程实践中，我们会遇到一些需要对泛型做进一步约束的场景。

类型约束完整定义是：适用在泛型函数和泛型类型上的类型，被强制约束为某种特定类型。类型约束指定了一个必须继承指定类的类型参数，或者遵循一个特定的协议。

	func findStringIndex<T: Equatable>(array: T[], valueToFind:T) -> Int? 
	{
	
	}

<T: Equatable> 意味着泛型参数指代的对象需要遵守Equatable协议。

### 更详细的约束－Where 语句

类型约束，对泛型参数的类型做了一定约束，可以强制要求泛型参数代表的类型遵守某个协议。而where语句可以更进一步对类型约束中声明的泛型参数所需要遵守的协议作出更详细的要求。where语句可以对协议的关联类型作进一步约束。比如，你可以要求两个泛型参数所遵守的协议的关联类型是相同的。

	//定义了关联类型的协议
	protocol Container {
    	typealias ItemType
    	mutating func append(item: ItemType)
    	var count: Int { get }
    	subscript(i: Int) -> ItemType { get }
	}

	//使用了Where约束的泛型函数
	func allItemsMatch<
    	C1: Container, C2: Container
    	where C1.ItemType == C2.ItemType, C1.ItemType: Equatable>
    	(someContainer: C1, anotherContainer: C2) -> Bool {
        
        // check that both containers contain the same number of items
        if someContainer.count != anotherContainer.count {
            return false
        }
        
        // check each pair of items to see if they are equivalent
        for i in 0..<someContainer.count {
            if someContainer[i] != anotherContainer[i] {
                return false
            }
        }
        
        // all items match, so return true
        return true    
	}

allItemsMatch函数的泛型约束有以下四条：

- C1和C2都需要遵守Container协议；
- C1的关联类型itemType和C2的关联类型必须相同；
- C1的关联类型遵守Equatable协议；
- 由于C1的关联类型和C2的关联类型相同，所以C2的关联类型也会遵守Equatable协议。

### 泛型与Any，AnyObject

Swift中有一个和范型类似的概念，Any和AnyObject。Any和AnyObject可用于表示任何值。

- AnyObject代表任意class类型的实例；
- Any可以代表任意类型，除了class，还包括struct，enum等在内的所有类型；

Swift会存在这两个东西有一定历史原因，这里不去深究。我们的关注点在泛型和Any的区别。它们都可以用于代表任何类型。区别在于，泛型提供了类型检查，而Any其实是回避了类型检查。

使用泛型定义一个函数：

	func generic<T>(x: T) -> T {
		return x	
	}

使用Any定义一个函数：

	func any(x: Any) -> Any {
		return x
	}

使用泛型，意味着编译器会做类型检查，函数的输入参数和返回值可以是任意类型，但是它们需要时同一类型。在Any的例子中，输入参数和返回值可以是不同的类型。

这个例子是为了表达，使用泛型，可以在写出更为灵活的代码的同时，保证类型安全。使用Swift时，应该避免使用Any，AnyType。

## 总结

Swift提供了全面的泛型编程语法，让你可以写出抽象层次更高，更为灵活的代码，避免了重复的代码的同时，又能拥有良好的类型安全性。
