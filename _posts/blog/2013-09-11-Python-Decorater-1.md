---
layout:    post
title:     Python 装饰器介绍（一）
category:  blog
description: 这项令人惊叹的功能终于出现了，伴随它的是一些关于其是否有用的关注。
tags: Python
---

我预言在某个时候装饰器将成为 Python 语言最重要的功能之一。问题是所有我所看到的关于装饰器的介绍都相当的令人迷惑，所以我将试图在这里纠正。


##装饰器 vs. 装饰器模式
首先，你需要理解我在使用“装饰器”这个词时带着些许不安。因为，我觉得它会很容易和[设计模式](http://www.amazon.com/gp/product/0201633612/ref=ase_bruceeckelA/)中的装饰器模式搞混。从某种角度看可能别的词也适用于这个特性，但 "decorator" 仍是不二之选。

确实，你可以使用 Python 装饰器来解释装饰器模式，但是这只是 Python 装饰器最局限的使用，我认为，Python 装饰器其实相当于宏（macros）。


##宏的由来
宏的历史并不短，不过多数人大概都是使用过C的预处理宏。C中宏的问题是：(1) 它们属于另外一个语言(不是C)；(2) 其行为有时候很古怪。并且经常和其余C的行为有不一致的情况。

Java和C#都加入了annotations机制，其允许用户为各程序变量进行标注。它们的问题是：(1) 为了实现一些东西，你有时不得不忍受方方面面的“磨练”，和 (2) annotation的特性因为这些语言天生的规范条款([Martin Fowler文绉绉地称之为“导向”](http://martinfowler.com/bliki/SoftwareDevelopmentAttitude.html))而受到局限。

除此之外，很多C++程序员（也包括我）也领教过C++模版的生成能力，并多少使用过一些这个和宏相似的特性。

很多其他语言都有过宏。虽然对其知之不多，但我敢肯定，Python decorators与Lisp宏在能力上不分伯仲。


##为什么使用宏
我想，从保守一点的角度说,宏在编程语言中的作用是提供了一种修改程序变量的方法。这也是Python中的decorators的工作--它们修改函数、若对class decorators则是整个类。这也是为何它们常常为metaclasses(元类)提供一个更简单的选择。

多数语言的自修正方法的主要失败之处是它们限制太多了，并且需要另外一种语言(我并不是说Java的annotations为了造就一个与众不同的annotation而使人饱经磨练，它就算包含了“另外一种语言”)。

Python被Fowler归入了“授权(enabling)”语言的行列，所以你如果想实现修正，为何还去创建一种与之不同或限制更多的语言？为何不使用Python本身呢？这恰恰是Python decorators的使命。


##Decorators的用途
Decorators允许在函数和类中嵌入或修改代码。这听起来和Java的Aspect-Oriented Programming (AOP,面向方面编程)挺像的，对吧？除此之外前者其实更简单，也更加强大。例如，假设你想在一个函数的入口和出口处做点手脚(比如做一些安全、跟踪和锁等--一切AOP标准变量)。使用decorators实现代码如下：

	@entryExit
	def func1():
		print "inside func1()"
	
	@entryExit
	def func2():
		print "inside func2()"

'@' 表明了是 decortor 程序。


##函数Decorators
一个函数decorators用于函数定义，它位于在函数定义之前的一行。例如：
	
	@myDecorator
	def aFunction():
		print "inside aFunction"

当编译器经过这段代码时，aFunction()被编译然后将结果函数对象传递给myDecorator代码，后者创建一个类函数对象并取代原来的aFunction()。

myDecorator代码长什么样呢？大多数介绍性的例子都将其作为函数给出，但我发现对于decoration机制使用类而非函数来理解decorators会更容易些，而且也更强大。

Decorator所返回的对象唯一的约束是它可以作为函数使用--也就意味着它必须是可调用的。因此，作为decorators使用的任何一个类必须实现__call__。

decorator该做什么呢？它什么都能做，只是你常常期望在某些地方能够使用原来的函数代码。其实没有这个必要：

	class myDecorator(object):
 	
    	def __init__(self, f):
        	print "inside myDecorator.__init__()"
        	f() # Prove that function definition has completed
 
	    def __call__(self):
    	    print "inside myDecorator.__call__()"
 
	@myDecorator
	def aFunction():
    	print "inside aFunction()"
 
	print "Finished decorating aFunction()"
 
	aFunction()

当运行这段代码时，会看见：

	inside myDecorator.__init__()
	inside aFunction()
	Finished decorating aFunction()
	inside myDecorator.__call__()

注意，myDecorator的构造器(constructor)在函数的decoration处执行。由于我们可以在 __init__() 里面调用f()，它意味着在调用decorator前就完成了f()的创建。另外需注意，decorator 构造器接收到 decorated 的函数对象。你将在构造器中得到函数对象，之后在 __call__() 方法中进行使用(当使用类时，decoration 和调用是两个泾渭分明的阶段，这也是我为何说它更简单、更强大的原因)。

当decorated之后再调用 aFunction()，它的行为就完全不一样了：不再用原来的代码而开始调用myDecorator.__call__() 方法。原因是 decoration 过程是将 decoration 结果取代原先的函数--在我们的例子中，myDecorator对象取代了aFunction。实际上，加入decorators之前，为了达到同样效果你需要编写十分晦涩的代码：

	def foo():pass
	foo = staticmethod(foo)
	
有了 '@' 这个 decoration 操作符的加入，你可以这么做：

	@staticmethod
	def foo():pass

这也是有人为何反对 decorators 的原因。因为‘@’实在有点像一块语法糖(syntax sugar,意指指那些没有给计算机语言添加新功能，而只是对人类来说更“甜蜜“的语法。语法糖往往给程序员提供了更实用的编码方式，有益于更好的编码风格，更易读。不过其并没有给语言添加什么新东西。--译者注)：通过另外一个函数传递函数对象，然后将结果赋给原来的函数。

我认为，decorator 之所以产生如此大影响，就是因为其带着一点点语法糖的味道改变了人们思考程序设计的方式。事实上，当成一种语言概念对其进行形式化，这种方法使“将代码应用到其他代码之中”(比如宏)的思想出现在主流思想之中。


##更多用处
现在，让我们回过头来实现第一个例子。这里我们实现一些更常规的东西，并在 decorated 函数中实际使用这些代码：

	class entryExit(object):
 
    	def __init__(self, f):
        	self.f = f
 
	    def __call__(self):
    	    print "Entering", self.f.__name__
        	self.f()
        	print "Exited", self.f.__name__
 
	@entryExit
	def func1():
    	print "inside func1()"
 
	@entryExit
	def func2():
    	print "inside func2()"
 
	func1()
	func2()

输出是：

	Entering func1
	inside func1()
	Exited func1
	Entering func2
	inside func2()
	Exited func2

现在可以看见，在整个调用期间 decorated 函数拥有了 “Entering” 和 “Exited” 跟踪语句。

构造器保存着自变量，即函数对象。在调用中，我们使用函数的 __name__ 属性来显示函数名称，然后调用函数本身。


###作为Decorators使用函数
对于一个 decorator 结果，其唯一的约束是必须是可调用的。这样它就可以完全取代 decorated 函数。在上面的例子中，我用包含一个 __call__() 方法的类的一个对象替代原有的函数。但函数对象仍是可调用的。因此我们可以使用函数(而非类)重写上一个例子，即：

	def entryExit(f):
    	def new_f():
        	print "Entering", f.__name__
       	 	f()
        	print "Exited", f.__name__
	    return new_f
 
	@entryExit
	def func1():
    	print "inside func1()"
 
	@entryExit
	def func2():
    	print "inside func2()"
 
	func1()
	func2()
	print func1.__name__

new_f() 在 entryExit() 里定义，所以在调用 entryExit() 时它就创建并返回了。注意，new_f() 是一个闭包(closure)，因为它获得的是 f 的实际值。

只要定义过 new_f()，它就从 entryExit() 返回，这样 decorator 机制就可以将结果作为 decorated 函数进行赋值了。

‘print func1.__name__’ 一行的输出是 new_f，因为 new_f 函数在 decoration 期间已取代原函数。如果还有疑问，可以在函数返回前改变 decorator 函数的名字：

	def entryExit(f):
    	def new_f():
        	print "Entering", f.__name__
        	f()
        	print "Exited", f.__name__
    	new_f.__name__ = f.__name__
    	return new_f

动态获得有关函数的信息、对函数的修改能力，都是Python的强大之处。


##更多例子
既然你有一定基础了，在[这里](http://wiki.python.org/moin/PythonDecoratorLibrary)可以看到更多一些decorators 的例子。注意使用类作为 decorators 的例子数量远多于使用函数的例子。

在本文中我有意避开含有自变量的 decorated 函数，在下一篇文章中我将重点讨论它。

##来源

---

**原文链接**：[链接](http://www.artima.com/weblogs/viewpost.jsp?thread=240808)
[连接](http://blog.csdn.net/beckel/article/details/3585352)
