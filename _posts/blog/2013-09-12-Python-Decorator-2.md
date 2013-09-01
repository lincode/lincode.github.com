---
layout:    post
title:     Python 装饰器参数（二）
category:  blog
description: 当向装饰器传入参数后，装饰器的行为有了巨大变化
tags: Python
---

（本文是 [Python 3 Patterns & Idioms](http://www.artima.com/weblogs/viewpost.jsp?thread=239183)（Python3之模式和用法） 一书的章节节选第二部分，点击这里阅读第一部分）


##回顾：不含参数的decorators
在前文中，我介绍了如何使用不含参数的 decorators，并使用类来实现。因为我发现这样做更容易接受。

如果创建了一个无参 decorator，被 decorated 的函数被传至构造器，每次调用 decorated 函数时就会调用 __call__() 方法：

	class decoratorWithoutArguments(object):
	
    	def __init__(self, f):
        	"""
        	If there are no decorator arguments, the function
        	to be decorated is passed to the constructor.
	        """
    	    print "Inside __init__()"
        	self.f = f

	    def __call__(self, *args):
    	    """
        	The __call__ method is not called until the
        	decorated function is called.
	        """
    	    print "Inside __call__()"
        	self.f(*args)
        	print "After self.f(*args)"

	@decoratorWithoutArguments
	def sayHello(a1, a2, a3, a4):
    	print 'sayHello arguments:', a1, a2, a3, a4

	print "After decoration"

	print "Preparing to call sayHello()"
	sayHello("say", "hello", "argument", "list")
	print "After first sayHello() call"
	sayHello("a", "different", "set of", "arguments")
	print "After second sayHello() call"

decorated函数的所有参数会被传至__call__()。输出结果是：

	Inside __init__()
	After decoration
	Preparing to call sayHello()
	Inside __call__()
	sayHello arguments: say hello argument list
	After self.f(*args)
	After first sayHello() call
	Inside __call__()
	sayHello arguments: a different set of arguments
	After self.f(*args)
	After second sayHello() call

注意， __init__() 是唯一一个被调用执行 decoration 的方法，每次调用 decorated 的 sayHello() 时就会调用__call__()。


##含有参数的decorators
现在让我们来修改上面的代码，看看向decorator加入参数后结果是什么。

	class decoratorWithArguments(object):
 
    	def __init__(self, arg1, arg2, arg3):
        	"""
    	    If there are decorator arguments, the function
	        to be decorated is not passed to the constructor!
        	"""
        	print "Inside __init__()"
        	self.arg1 = arg1
        	self.arg2 = arg2
        	self.arg3 = arg3
 
	    def __call__(self, f):
    	    """
        	If there are decorator arguments, __call__() is only called
       		once, as part of the decoration process! You can only give
        	it a single argument, which is the function object.
	        """      
		   	print "Inside __call__()"    
    	    def wrapped_f(*args):
        	    print "Inside wrapped_f()"
            	print "Decorator arguments:", self.arg1, self.arg2, self.arg3
            	f(*args)
            	print "After f(*args)"
	        return wrapped_f

	@decoratorWithArguments("hello", "world", 42)
	def sayHello(a1, a2, a3, a4):
    	print 'sayHello arguments:', a1, a2, a3, a4

	print "After decoration"

	print "Preparing to call sayHello()"
	sayHello("say", "hello", "argument", "list")
	print "after first sayHello() call"
	sayHello("a", "different", "set of", "arguments")
	print "after second sayHello() call"

从输出结果可以看到，加入参数使程序执行发生了很大变化。

	Inside __init__()
	Inside __call__()
	After decoration
	Preparing to call sayHello()
	Inside wrapped_f()
	Decorator arguments: hello world 42
	sayHello arguments: say hello argument list
	After f(*args)
	after first sayHello() call
	Inside wrapped_f()
	Decorator arguments: hello world 42
	sayHello arguments: a different set of arguments
	After f(*args)
	after second sayHello() call

现在 decoration 方法调用构造器，然后就马上调用 __call__()，后者只能包含一个参数（函数对象）且返回替代原有函数的 decorated 函数对象。注意当前 decoration 期间 __call__() 仅被调用一次，此后从 __call__() 返回的 decorated 函数就可以在实际调用中使用了。

虽然这种机制有一定合理性：构造器在这里可获取 decorator 参数，但 __call__() 对象不能再作为 decorated 函数使用了。因此你必须使用 __call__() 执行 decoration ：可能第一次遇到这种与无参情况截然不同的方式你会比较吃惊，何况还必须编写和无参 decorator 完成不同的代码。

##含decorator参数的decorator函数
最后，让我们看一个更复杂一点的 decorator 函数实现，它需要你处理所有细节：

	def decoratorFunctionWithArguments(arg1, arg2, arg3):
    	def wrap(f):
        	print "Inside wrap()"
        	def wrapped_f(*args):
            	print "Inside wrapped_f()"
            	print "Decorator arguments:", arg1, arg2, arg3
            	f(*args)
            	print "After f(*args)"
        	return wrapped_f
	    return wrap
 
	@decoratorFunctionWithArguments("hello", "world", 42)
	def sayHello(a1, a2, a3, a4):
    	print 'sayHello arguments:', a1, a2, a3, a4
 
	print "After decoration"
	
	print "Preparing to call sayHello()"
	sayHello("say", "hello", "argument", "list")
	print "after first sayHello() call"
	sayHello("a", "different", "set of", "arguments")
	print "after second sayHello() call"

输出结果为：

	Inside wrap()
	After decoration
	Preparing to call sayHello()
	Inside wrapped_f()
	Decorator arguments: hello world 42
	sayHello arguments: say hello argument list
	After f(*args)
	after first sayHello() call
	Inside wrapped_f()
	Decorator arguments: hello world 42
	sayHello arguments: a different set of arguments
	After f(*args)
	after second sayHello() call

decorator 函数的返回值必须是一个封装待 decorated 函数的函数。也就是说，Python 会保存返回函数然后在 decoration 期间调用，并传递待 decorated 函数。这也是为何有三层函数的原因：里面那个函数才是被替换的。

由于闭包，wrapped_f() 有权访问 decorator 参数 arg1, arg2 和 arg3，而无需像在 class 版本中那样显式存储它们。然而，我也是在这里发现了“显胜于隐（explicit is better than implicit）”。即使该函数版本看起来要更加简洁紧凑，但我发现还是类版本容易理解，当然也就容易修改和维护。

###下一节内容
在下一节中我会给出 decorators 的一些实例 －－ 基于 Python 开发的 build system －－ 然后在最后一节讨论类 decorators。

##来源

---

**原文链接**：[链接](http://www.artima.com/weblogs/viewpost.jsp?thread=240845)
[连接](http://blog.csdn.net/beckel/article/details/3945147)
