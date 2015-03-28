---
layout:     post
title:      Swift函数式编程－不变性
category: blog
description: Swift支持函数式编程，这一篇介绍不变性（immutable）
tags: blog
---

Swift支持函数式编程，这一篇介绍不变性（immutable）。

### 不变性

不变性是函数式编程的基础。

先讨论一下Haskell这类纯函数式语言。简单而言，Haskell没有变量。这是因为，Haskell追求更高级别的抽象，而变量其实是对一类低级计算机硬件：存储器空间（寄存器，内存）的抽象。变量存在的原因，可以视为计算机语言进化的遗迹，比如在初期直接操作硬件的汇编语言中，需要变量来使用操作存储过程。而在计算机出现之前，解决数学计算问题都是围绕构建数学函数。数学中，不存在计算机语言中这种需要重复赋值的变量。

而Haskell则基于更抽象的数学模型。使用Haskell编程只需专注于设计数据之间的映射关系。而在数学上，表示两个数据之间映射关系的实体就是函数。这使得编写Haskell代码和设计数学函数的过程是一致的，Haskell程序员的思路也更接近数学的本质。

Haskell摒弃了变量的同时，也抛弃了循环控制。这是因为没有变量，也就没有了控制循环位置的循环变量。这也很好理解。回忆一下我们在学习计算机之前的数学课程中，也无需使用到for这类概念。我们还是使用函数处理一个序列到另外一个序列的转换。

Swift提供了一定程度的不变性。在Swift中，被声明为不变的对象在完成对其初始构造之后就不可改变。换句话说，构造器是唯一个可以改变对象状态的地方。如果你想改变一个对象的值，只能使用修改后的值来创建新的对象。

不变性是为了减少或者消灭状态。面向对象编程语言中，状态是计算的基础信息。如何可控地修改状态，Java，Ruby等编程语言都给出了大量的语言机制，比如，可见性分级。但是，由于大量可变状态的存在，使用面向对象编程语言在编写高并发，多线程代码时会有很多困难。因为，你无法知道并行进行的诸多状态读写中是否有顺序上的错误。而且这种错误又是难以察觉的。而不变性解决了这个问题。不变性意味函数没有副作用，无论多少次执行，相同的输入就意味着相同的输出。那么，多线程环境中就没有了烦人的同步机制。所有线程都可以无所顾忌的执行同一个函数的代码。

而在Java这类面向对象编程语言中，变量用于表示对象本身的状态。Swift作为支持多种范型的编程语言，即支持变量，也支持方便地申明不变量。

### 变量和不变量


Java中，声明不变量：
	
	#变量
	private string mutable;
	#不变量
	private final String immutable;

Scala中，声明不变量：

	#变量
	var mutable
	#不变量
	val immutable = 1
	
Swift中声明变量和不变量：

	#变量
	var mutable
	#不变量
	let immutable ＝ 1

Swift中声明了不变量，就必须在声明时同时初始化，或者在构造器中初始化。这两个地方之外，就无法再改变不变量了。Swift区分`var`和`let`不仅仅是为了区分变量和不变量，同时也是为了使用编译器来强制这种区分。声明不变量是受到鼓励的。因为，使用不变量更容易写出，容易理解，容易测试，松耦合的代码。

### 不可变类

由于不可变性具有例如线程安全性这类天生优势，在编写面向对象语言时，我们也会有使用到不变对象的场景。但由于编程范式不同的原因，在面向对象语言中构造不可变类是一件非常麻烦的事情。

以Java为例，如果将一个类构造成不可变的类，需要做如下事情：

- 将类声明为final。这样就不能继承该类。无法继承该类，就无法重写它的方法的行为。Java 中的String 类就使用了这种策略。

- 所有的实例变量都声明为final。这样，你就必须在申明时初始化它，或者在构造器中初始化它们。在其他地方，你都将无法改变声明为final的实例变量。

- 提供合适的构造过程。对于不可变类，构造器是唯一可以初始化它的地方。所以，提供一个合适的构造器是实用不可变类的必要条件。

- 除构造器之外不提供任何可以改变状态的方法。实例变量被声明为final，其实就已经无法改变它们了。但是，仍然可以改变它们所指向的内容。因此，在 getter 方法中考虑防御方法：不直接返回内容的引用，而是返回复制内容的引用。

一个Java实现的不可变类的例子如下：

	public final class Person {
		private final String name;
		private final List<String> interests;

	    public Person(String name, List<String> interests) {
        	this.name = name;
        	this.streets = streets;
        	this.city = city;
    	}

    	public String getName() {
        	return name;
    	}    	

    	public List<String> getInterests() {
        	return Collections.unmodifiableList(interests);
    	}
	}

具有函数特性的多范式编程语言中，大多数会为构造不变类提供方便。比如Groovy提供了`@Immutable`注释来表示不可变类。

	@Immutable
	class Preson {
    	String name
    	String[] interests
	}

`@Immutable` 提供了以下功能：

- 它是final的，即不可被继承的；
- 属性自动拥有了私有的，并且自动产生了getter方法；
- 任何改变属性的企图都会导致抛出 ReadOnlyPropertyException 异常；
- Groovy创建了合适的构造函数：即创建了有序的构造函数，又创建了基于映射的构造函数；
- 集合类被封装在适当的包装器中，数组（及其他可克隆的对象）被克隆。
- 自动生成默认的 equals、hashcode 和 toString 方法。

Swift实现一个不可变类的方法的例子：

	struct Person {
		let name:String
		let interests:[String]
	}

- 结构体（struct）是final的，即不可被继承；
- `let` 声明的实例变量，保证了类初始化之后，实例变量无法再被改变；
- struct 是值类型，将一个struct赋给另外一个变量，其实是拷贝了对像，将拷贝的对象赋值给另一个变量。

Swift中实现一个不可变的类的方法是：声明一个结构体（`struct`），并将该结构体的所有实例变量以`let`开头声明为不变量。在不变性这方面，枚举（`enum`）具有和结构体相同的特性。所以，上面例子中的结构体在合适的场景下，也可以被枚举类型替换。

### 值类型和引用类型

￼值类型在赋值和作为函数参数的时候被传递给一个函数的时候，实际上操作的是其的拷贝。Swift中有大量值类型，包括数字，字符串，数组，字典，元组，枚举和结构体等。

	struct PersonStruct {
		var name:String
	}

	var structPerson = PersonStruct(name:"Totty")
	var sameStructPerson = structPerson
	sameStructPerson.name = "John"
	print(structPerson.name)
	print(sameStructPerson.name)

	// result:
	// "Totty"
	// "John"

可以看到，structPerson和sameStructPerson的值不一样了。在赋值的时候，sameStructPerson的到是structPerson的拷贝。

引用类的实例 (主要是类) 可以有多个所有者。在赋值和作为函数参数的时候被传递给一个函数的时候，操作的是其引用，而并不是其拷贝。这些引用都指向同一个实例。对这些引用的操作，都将影响同一个实例。

	class PersonClass {
		var name:String
	}

	var classPerson = PersonClass(name:"Totty")
	var sameClassPerson = structPerson
	sameClassPerson.name = "John"
	print(classPerson.name)
	print(sameClassPerson.name)

	// result:
	// "John"
	// "John"

可以看到，sameClassPerson的改变，同样也影响到了classPerson。其实它们指向同一个实例。这种区别在作为函数参数时也是存在的。

在Swift中区分值类型和引用类型是为了让你将可变的对象和不可变的数据区分开来。Swift增强了对值类型的支持，鼓励我们使用值类型。使用值类型，函数可以自由拷贝，改变值，而不用担心产生副作用。

### 纯函数

不变性导致另外一个结果，就是纯函数。纯函数即没有副作用的函数，无论多少次执行，相同的输入就意味着相同的输出。这是非常好的特性。它意味着理解起来更简单，测试起来更方便，线程安全性。

### Objective-C中的不变性

Objective-C中，苹果的Foundation库提供了不少具有不变性的类：NString相对于NSMutableString，NSArray相对于NSMutableArray，以及NSURL等等。在Objective-C中，绝大多数情况下，使用不变类是缺省选择。但是，Objective－C中没有如Swift中`let`这样简单强制不变性的方法。

### 总结

不变性的好处:

- 更高层次的抽象。程序员可以以更接近数学的方式思考问题。

- 更容易理解的代码。由于不存在副作用，无论多少次执行，相同的输入就意味着相同的输出。纯函数比有可变状态的函数和对象理解起来要容易简单得多。你无需再担心对象的某个状态的改变，会对它的某个行为（函数）产生影响。

- 更容易测试的代码。更容易理解的代码，也就意味着测试会更简单。测试的存在是为了检查代码中成功发生的转变。换句话说，测试的真正目的是验证改变，改变越多，就需要越多的测试来确保您的做法是正确的。如果你能有效的限制变化，那么错误的发生的可能就更小，需要测试的地方也就更少。变化只会发生构造器中，因此为不可变类编写单元测试就成了一件简单而愉快的事情。

- 线程安全的代码。这意味着多线程环境下，运行代码没有同步问题。它们也不可能因为异常的发生而处于无法预测的状态中。

不像Haskell这种纯函数式编程语言只能申明不可变量，Swift提供变量和不可变量两种申明方式。这使得程序员有选择的余地：在使用面向对象编程范式时，可以使用变量。在需要的情况下，Swift也提供不变性的支持。 
