---
layout:    post
title:     Effective Objective-C 2.0 Chapter 2
category:  blog
description: 对象，消息，和运行时
tags: ObjC
---
# 第2章 对象，消息，和运行时
对象在诸如 Objective-C 这样的面向对象语言中是程序的构件，提供了一个存储搬运数据的方法。消息是对象间相互对话以搬运数据，产生行为的过程。深入理解对象和消息这两个特性是构建高效和可维护代码的关键。

Objective-C 运行时是应用运行时，才提供给语言使用的代码。运行时提供使得在对象间发送消息成为可能，和类的实例如何创建背后的所有逻辑的这样的关键功能。理解这些能使你成为一个更好的开发者。

## 条目 6：理解属性（Properties）

### 要点回顾
属性（Properties）是一个提供了对象的数据进行封装的 Objective-C 功能。对象在 Objective-C 中通常包含一组用于存储数据的实例变量，类会在这些数据上工作。实例变量通常通过访问器方法来访问。取值器用于读取变量，赋值器用于写变量。这个概念已经标准化并成为在 Objective-C 2.0 中发布的一个被成为属性的功能。属性允许开发者告知编译器自动地写访问器。这个功能通过一个新语法“点”更简单地访问被类存储地数据。你可能已经使用了属性，但你可能并不知道所有的选项。 你也可能没有意识到围绕属性的复杂性。条目 6 说明了属性解决了那些问题的背景和属性的关键功能。

一个类描述了一个人，可能存储了人名，生日，住址邓。你如下面一个，可以在公开接口（public）中为类声明实例变量：

	@interface EOCPerson : NSObject {
	@public
    	NSString *_firstName;
    	NSString *_lastName;
	@private
    	NSString *_someInternalData;
	}
	@end

如果你来自 Java 或者 C++ 世界，定义实例变量的可见性，对于你应该会很熟悉。可是，这个技术在 Objective-C 中很少用到。这个方法的问题是一个对象的布局在编译时就已经定义好了。无论变量 _firtstName 何时被访问，编译器都会将偏移量硬编码到对象的存储的内存区域。这工作得很好直到你添加了其它实例变量。例如，假设另一个实例变量被加到 _firstName 之前。

	@interface EOCPerson : NSObject {
	@public
    	NSDate *_dateOfBirth;
    	NSString *_firstName;
    	NSString *_lastName;
	@private
    	NSString *_someInternalData;
	}
	@end

曾经指向 _firstName 得偏移量现在指向了 _dateOfBirth。任何获得硬编码过的偏移量的代码都可以最终读取到错误的值。为了展示这一点，图 2.1 显示了假设指针为 4 字节，在添加 _dateOfBirth 之前和之后的类的内存分布。

![alt property](/images/blog/EffectiveObjC/Property-2-1.png "Property 2-1")

**图 2.1** 添加一个实例变量之前和之后的类数据的分布

使用编译器计算出来的偏移量的代码在类定义改变之后将会被破坏，除非重新编译它。例如，某个库的代码可能还使用着旧的类定义。如果链接的代码使用的却是新的类定义，这将在运行时产生不兼容。为了克服这个问题，各类语法都发明了很多技术。Objective-C 的采用的方法是使类实例变量成为被类对象（见条目 14 以更多了解类对象）持有的特殊变量以存储偏移量。然后在运行时，偏移量会被查询，如果类定义变化了，存储的偏移量就会被更新；无论何时访问实例变量，正确的偏移量都会被用到。你甚至能在运行时将实例变量加入类。这个方法被成为非脆弱应用二进制接口（ABI）。ABI 在各部分之间约定了代码应该如何生成的相关规则。非脆弱 ABI 也意味着实例变量可以被定义在类的目录（见条目 27）中，或者在实现文件中。所以你不再需要将所有实例变量声明在接口中，你因此可以不在公开接口中泄露关于实现的内部信息。

鼓励使用访问器方法，而不是直接使用实例变量是另外一种克服这个问题的方法。属性背靠实例变量，胆识它们提供了一个干净的抽象。你可以自己写访问器，但应该以正确的 Objective-C 风格，访问器遵循严格的命名规范。因为严格的命名，它是可能成为一个提供自动创建访问器方法的语言结构。这就是 @property 语法引入的原因。

你使用属性定义对象的接口，提供访问被对象封装的标准方法。因此，属性也可以被认为是告知这有个可以访问给定类型和给定名字的变量的访问器的便捷方法。例如，考虑如下类：

	@interface EOCPerson : NSObject
	@property NSString *firstName;
	@property NSString *lastName;
	@end

对于类的使用者，写如下类和上面是等价的：

	@interface EOCPerson : NSObject
	- (NSString*)firstName;
	- (void)setFirstName:(NSString*)firstName;
	- (NSString*)lastName;
	- (void)setLastName:(NSString*)lastName;
	@end

为了使用属性，你可以使用点操作符，类似在纯 C 中访问一个栈中分配的结构体成员。比阿拟器将点操作符转化为访问器调用，就像你直接调用它们一样。因此，使用点和直接调用方法之间没有区别。下面代码表现了这种等价性。

	EOCPerson *aPerson = [Person new];

	aPerson.firstName = @"Bob"; // Same as:
	[aPerson setFirstName:@"Bob"];

	NSString *lastName = aPerson.lastName; // Same as:
	NSString *lastName = [aPerson lastName];

但是属性不仅仅就只有这个作用。如果你愿意，通过一个被称为自动综合的过程编译器可以自动为你生成这些方法的代码。必须提出是编译器在编译时完成这些的，你在编辑器里是看不到综合出的方法的源代码的。根据生成的代码，编译器也自动向类添加了特定类型和名字，以下划线为前缀的实例变量。在前面的例子中，会有两个实例变量：_firstName 和 _lastName。也可以通过在实现文件中使用 @synthesize 语法控制实例变量的名字，如下：

	@implementation EOCPerson
	@synthesize firstName = _myFirstName;
	@synthesize lastName = _myLastName;
	@end

使用上面语法将产生名为 _myFirstName 和 _myLastName 而不是缺省名的实例变量。通常不会改变实例变量的缺省名；可是，如果你不是一个使用下划线法命名实例变量的爱好者，你可以仍可以使用这个方法给他们命你想要的名字。胆识我鼓励你使用缺省的名字，如果每个人都坚持相同的约定，这会使得代码对每个人都易读。

如果你不想要编译器综合出访问器方法，你可以自己实现方法。可是，如果你只实现一个访问器，编译器还是为你会综合出其它的访问器。另一种方法是使用 @dynamic 关键字停止综合，这个关键字高数编译器不要自动创建 property 背后的实例变量和访问器。同时，当编译中的代码访问属性时，编译器将会忽略访问器还没被定义的事实，它将相信这些访问器在运行时可用。例如，这被用于继承 CoreData 的 NSManagedObject，访问器在运行时被动态创建。NSMangedObject 选择这个方法是因为属性不是实例变量。而来自后端数据库的数据将被用于属性值输出。例如：

	@interface EOCPerson : NSManagedObject
	@property NSString *firstName;
	@property NSString *lastName;
	@end

	@implementation EOCPerson
	@dynamic firstName, lastName;
	@end

在这个类中，没有访问器或者实例方法会被综合。如果你试图访问属性，编译器也不会给出警告。

### 属性的参数
属性其余你应该注意的地方时所有其所有可用于控制编译器如何生成属性的参数。例如，如下例子使用了三个参数：

	@property (nonatomic, readwrite, copy) NSString *firstName;

有四种类型的参数可以使用：

#### Atomicity
综合出来的访问器缺省地包含了锁定使其访问原子化。如果你使用参数 nonatomic，则不会使用锁。注意虽然没有 atomic 参数（原子化被假设为没有 nonatomic 都为原子化的），但在你要显式显示时，atomic 仍然可以被应用而不会出现编译错误。如果你自己定义访问器，你需要自己做特别的原子化。

#### Read/Write
* readwrite 取值器和赋值器都可用。如果属性被综合了，编译器将生成两个方法。
* readonly 只有取值器可用，如果属性被综合了，编译器将只生成取值器。如果你只向外部暴露属性的读权限，你可能会需要用到这个。在类的延续目录中重新声明其为可读写。见条目 27 以获取更多信息。

#### Memory-Management Semantics
属性封装数据，数据需要具体他们自己的语义。这只在赋值器中产生影响。例如，赋值器应该保留新值，还是简单地将相关实例变量取出？当编译器综合访问器时，它用这些参数决定应该为你写什么样地代码。如果你创建自己地访问器，你必须自己搞定这些参数指定的特性。

* assign 赋值器是一个简单的赋值操作，用于纯量类型，例如 CGFloat 或者 NSInteger。
* strong 这指明了属性定义了一个拥有关系。当赋新值时，它将先被保留，旧值将被释放，然后在赋值。
* weak 这指明了属性定义了一个非拥有关系。当赋新值时，它不会被保留，旧值也不会被释放。这类似于 assign 所作的，但是当属性指向的对象被销毁时，值都被置为 nill。
* unsafe_unretained 这和 assign 语义相同，但它用于指明数据类型是一个非拥有关系（unretained）的对象类型，当目标被销毁时，值并不象 weak 一样，被不会被置为 nil。
* copy 这指明了一个类似于 strong 的拥有关系；但是，值被拷贝，而不是被保留了。这常用于 NSString* 类型以保护封装性，因为传入赋值器的可能会是子类 NSMutableString* 的实例。如果是可变对象，值在赋值之后还可以可以在对象不知道的情况下变化。所以不可变拷贝被用于确定字符串不能在对象里变化。任何可变对象都应该接受 copy。

#### Method Names
访问器方法的名字可以被如下参数控制：

* getter=<name> 指定取值器的名字。这个方法常被用于布尔属性，你可能希望在取值器前加前缀 is 。例如，在类 UISwitch 中，表示开关是开是关的属性被如此定义：

	@property (nonatomic, getter=isOn) BOOL on;

* setter=<name> 指定赋值器的名字。这个方法不常用到。

你可以使用这些参数获得对于综合出来的访问器细粒度的控制。可是，你应该注意如果你自己实现你自己的访问器，你应该使访问器符合你自己指定的参数。例如，一个属性声明为 copy 旧应该确保赋值器中赋予的对象的拷贝。否则，属性的使用者将会处于错误的印象下，并且 bug 也将由于约定没有被遵守而产生。

即使是在其他方法中，确保你实现了属性定义中的正确的语义是很重要的。例如，考虑类 EOCPerson 的扩展。它将内存管理语义声明为 copy，因为值是可变的。它也加了一个设置名和姓的初始值的构造器：

	@interface EOCPerson : NSManagedObject

	@property (copy) NSString *firstName;
	@property (copy) NSString *lastName;

	- (id)initWithFirstName:(NSString*)firstName
    	           lastName:(NSString*)lastName;

	@end

在这个自己实现的构造器中，按照属性定义实现 copy 语义是很重要的。原因是属性定义相当于约定了类中的值如何被设置的文档。所以构造器的实现看起来将象这样：

	- (id)initWithFirstName:(NSString*)firstName
    	           lastName:(NSString*)lastName
	{
	    if ((self = [super init])) {
        	_firstName = [firstName copy];
        	_lastName = [lastName copy];
    	}
    	return self;
	}

你可能想知道你不能简单地用属性的赋值器替代上面 init 里的赋值语句，这将保证使用了正确的语义。但你不应该在 init（或者 dealloc 中）使用你自己的访问器，这将在条目 7 中解释。

如果你已读了条目 18，你将会知道，如果可能，最好使对象为不可变的。应用到 EOCPerson，你可以使两个属性都只读。构造器将赋值后，你就不能改变它们了。在这个场景中，声明内存管理语义是很重要的。所以属性定义最终应该看起来像这样：

	@property (copy, readonly) NSString *firstName;
	@property (copy, readonly) NSString *lastName;

即使因为属性是只读的，赋值器没有被创建，将构造器赋值时使用的语义文档化仍然很重要。没有这样的文档化，类的使用者将不能判断他已拿到了一个拷贝，因此可能会在调用构造器之前制作一个额外的拷贝。这么做是多余的无效的。

你可能像知道 atomic 和 nonatomic 之间的不同。如前面描述的，atomic 的访问器包含了锁以确保原子性。这意味着如果两个线程同时在读写相同属性时，属性的值在任何时刻都是有效的。没有锁，或者说 nonatomic，属性值在一个线程修改过程中，另外一个线程可能正在读。如果这个发生了，读取的值将是无效的。

如果你完全只为 iOS 做开发，你会看到所有的属性都被声明为 nonatomic。理由是，历史原因，锁引起的开销在 iOS 中是一个性能问题。通常，原子性不被要求，因为不能保证线程安全，这通常需要深层次的锁。例如，即使有原子性，单个线程可能连续多次读取属性却获得不同的值，如果另外一个线程正在同一时间修改属性的话。因此，通常在 iOS 中使用 nonatomic 属性。但是在 Mac OS X 中，你也不常能发现 atomic 属性，因为它是个性能瓶颈。


* @property 语法提供了一种定义对象封装何种数据的方法。

* 使用类变量为被存储的数据提供正确语言。
 
* 确定任何属性背后的实例变量被赋值了，声明语义都被实现了。

* 在 iOS 中使用 nonatomic，因为如果使用 atomic，性能会受严重影响。


## 条目 7：在对象内部访问实例变量，主要采用直接访问的形式

## 条目 8：理解对象相等

## 条目 9：使用类簇模式隐藏实现细节

## 条目 10：使用相关对象法将客户数据挂在已有类上

## 条目 11：理解 objc_msgSend 的角色

## 条目 12：理解消息发送

## 条目 13：考虑使用 Swizzling 方法调试黑箱方法

## 条目 14：理解对象类是什么