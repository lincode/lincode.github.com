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

使用编译器计算出来的偏移量的代码在类定义改变之后将会被破坏，除非重新编译它。例如，某个库的代码可能还使用着旧的类定义。如果链接的代码使用的却是新的类定义，这将在运行时产生不兼容。为了克服这个问题，各类语法都发明了很多技术。Objective-C 的采用的方法是使类实例变量成为被类对象（见条目 14 以更多了解类对象）持有的特殊变量以存储偏移量。然后在运行时，偏移量会被查询，如果类定义变化了，存储的偏移量就会被更新；无论何时访问实例变量，正确的偏移量都会被用到。你甚至能在运行时将实例变量加入类。这个方法被成为非脆弱应用二进制接口（ABI）。ABI 在各部分之间约定了代码应该如何生成的相关规则。非脆弱 ABI 也意味着实例变量可以被定义在类的分类（见条目 27）中，或者在实现文件中。所以你不再需要将所有实例变量声明在接口中，你因此可以不在公开接口中泄露关于实现的内部信息。

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

使用上面语法将产生名为 \_myFirstName 和 \_myLastName 而不是缺省名的实例变量。通常不会改变实例变量的缺省名；可是，如果你不是一个使用下划线法命名实例变量的爱好者，你可以仍可以使用这个方法给他们命你想要的名字。胆识我鼓励你使用缺省的名字，如果每个人都坚持相同的约定，这会使得代码对每个人都易读。

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
* readonly 只有取值器可用，如果属性被综合了，编译器将只生成取值器。如果你只向外部暴露属性的读权限，你可能会需要用到这个。在类的附加分类中重新声明其为可读写。见条目 27 以获取更多信息。

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

### 要点回顾

* @property 语法提供了一种定义对象封装何种数据的方法。

* 使用类变量为被存储的数据提供正确语言。
 
* 确定任何属性背后的实例变量被赋值了，声明语义都被实现了。

* 在 iOS 中使用 nonatomic，因为如果使用 atomic，性能会受严重影响。


## 条目 7：在对象内部访问实例变量，主要采用直接访问的形式

外部访问对象的实例参数应该总是使用属性，但是如何内部访问对象的实例参数是 Objective-C 社区内部热烈讨论的主题。一些人建议总是使用属性访问实例变量，一些人建议总是直接访问实例变量，一些人建议两种混合使用。我强烈建议你直接读取实例变量，但赋值时使用属性。但仍然有几点需要注意。考虑下面的类：

	@interface EOCPerson : NSObject
	@property (nonatomic, copy) NSString *firstName;
	@property (nonatomic, copy) NSString *lastName;

	// Convenience for firstName + " " + lastName:
	- (NSString*)fullName;
	- (void)setFullName:(NSString*)fullName;
	@end

便利方法 fullName 和 setFullName 可能被实现成这样：

	- (NSString*)fullName {
    	return [NSString stringWithFormat:@"%@ %@",
        	    self.firstName, self.lastName];
	}

	/** The following assumes all full names have exactly 2
     *  parts. The method could be rewritten to support more
	 *  exotic names.
	 */
	- (void)setFullName:(NSString*)fullName {
    	NSArray *components =
        	[fullName componentsSeparatedByString:@" "];
    	self.firstName = [components objectAtIndex:0];
    	self.lastName = [components objectAtIndex:1];
	}

在取值器和赋值器中，我们都使用属性的点操作符，通过访问器方法访问实例变量。现在假设你重写了这两个方法，改为直接访问实例变量：

	- (NSString*)fullName {
    	return [NSString stringWithFormat:@"%@ %@",
        	    _firstName, _lastName];
	}

	- (void)setFullName:(NSString*)fullName {
    	NSArray *components =
        	[fullName componentsSeparatedByString:@" "];
    	_firstName = [components objectAtIndex:0];
	    _lastName = [components objectAtIndex:1];
	}

这两种风格有些许不同：

* 直接访问实例变量无疑将更快一些，因为它不通过 Objective-C 的方法分发（见 条目 11）。编译器将生成直接访问对象的实例变量所存储的内存地址的代码。
* 直接访问实例撇开了由赋值器定义的属性的内存管理语义。例如，如果你的属性被声明为 copy，直接赋值实例变量将不会做拷贝操作。新值将被保留，旧值会被释放。
* 在直接访问实例变量时，键值观察（KVO）的提醒将不会被触发。这也许会产生问题，也许不会，这取决于你想要你的对象有何种行为。
* 通过属性访问，可使得调试属性的相关问题更为容易，因为你可以在访问器和赋值器中添加断点，以获知谁在什么时候访问了属性。

一个不错的折衷是给实例变量赋值时使用赋值器，而在读实例变量时则直接访问。这么做拥有了更快速的读操作，同时也没有失去通过属性控制写操作的好处。通过赋值器做写操作的最重要的理由是你确保了内存管理语义被贯彻了。可是，这个方法仍然有几个需要注意的地方。

第一个需要注意的是，当在构造器中赋值时。这时，你应该总是直接给实例变量赋值，因为子类可能会重写赋值器。考虑 EOCPerson 有一个被设计为表示有 Smith 这个姓的人的子类 EOCSmithPerson。这个子类可能会这样重写 lastName 的赋值器：

	- (void)setLastName:(NSString*)lastName {
	    if (![lastName isEqualToString:@"Smith"]) {
        	[NSException raise:NSInvalidArgumentException
            	        format:@"Last name must be Smith"];
    	}
    	self.lastName = lastname;
	}

基类 EOCPerson 可能在它的缺省构造器中赋给了姓空字符。如果它通过赋值器这么做，那么子类的赋值器将会被调用而抛出一个异常。然而，在一些情况下，在构造器中你必须使用赋值器。比如在父类中声明的实例变量；你无法直接访问实例变量，所以你必须使用赋值器。

另外需要注意的是当属性使用延迟实例化时。在这个例子中，你必须使用取值器；如果你不使用取值器，实例变量将永无机会被实例化。例如，类 EOCPerson 可能有一个属性，用于访问一个代表了每个人的大脑的复杂对象。这个属性不经常被访问，构造成本又很高，你也许会在取值器中如此延迟实例化它：

	- (EOCBrain*)brain {
    	if (!_brain) {
        	_brain = [Brain new];
    	}
    	return _brain;
	}

如果你直接访问实例变量，取值器不会被调用，brain 将不会被构造，你需要调用访问器已访问属性 brain。

### 要点回顾
* 推荐内部直接读取实例变量，和通过属性向实例变量写数据。

* 在构造器和 dealloc 中，总是直接读写实例变量。

* 有时，在数据被延迟初始化时，你会需要通过属性读取数据。


## 条目 8：理解对象相等

能够比较对象的相等性是非常有用的。可是，使用 == 操作符做比较通常并不是你想要，因为这么做是比较指针本身，而不是比较指针指向的对象。作为代替，你应该使用 NSObject 协议中声明的 isEqual: 方法来检查对象的相等性。通常，两个不同的类的对象总是不相等的。有些对象也提供了特别的相等性检查方法，你就可以使用这些方法，如果你已经知道两个你正在检查的对象是相同的类。例如，如下代码：

	NSString *foo = @"Badger 123";
	NSString *bar = [NSString stringWithFormat:@"Badger %i", 123];
	BOOL equalA = (foo == bar); //< equalA = NO
	BOOL equalB = [foo isEqual:bar]; //< equalB = YES
	BOOL equalC = [foo isEqualToString:bar]; //< equalC = YES

这里，你可以看到 == 和相等方法之间的不同。NSString 是实现了自己的相等性检查方法的类的例子，方法名为：isEqualToString:。传入的这个方法的对象也必须是一个 NSString 对象；否则，结果将会是不确定的。这个方法被设计为快于调用 isEqual:。isEqual 必须做一些额外的步骤，因为它不知道被比较的对象的是什么类。

在 NSObject 协议中有两个关于相等性检查的核心方法，它们是：

	- (BOOL)isEqual:(id)object;
	- (NSUInteger)hash;

这两个方法在 NSObject 类中缺省实现是，如果它们的指针值完全相等，两个对象才相等。为了了解如何为你自己的类重写，先理解协议是很重要的。任何两个对象使用 isEqual: 方法被确定相等，都必须从 hash 方法中返回相同的值。可是，两个从 hash 方法返回相同值的对象并不必须是通过 isEqual: 方法判断为相等的。

例如，考虑下面的类：

	@interface EOCPerson : NSObject
	@property (nonatomic, copy) NSString *firstName;
	@property (nonatomic, copy) NSString *lastName;
	@property (nonatomic, assign) NSUInteger age;
	@end

如果两个 EOCPerson 对象所有的变量都相等，则两个对象相等。所以 isEqual: 方法看起来应该像这样：

	- (BOOL)isEqual:(id)object {
    	if (self == object) return YES;
	    if ([self class] != [object class]) return NO;
	
    	EOCPerson *otherPerson = (EOCPerson*)object;
    	if (![_firstName isEqualToString:otherPerson.firstName])
        	return NO;
	    if (![_lastName isEqualToString:otherPerson.lastName])
    	    return NO;
    	if (_age != otherPerson.age)
        	return NO;
	    return YES;
	}
	
首先，object 被检查指向它们的指针等于 self。如果指针相等，对象必然是相等的，因为它们是同一个对象！然后，对比两个对象的类。如果类不相同，这两个对象不能相等。毕竟，一个 EOCPerson 不能对于一个 EOCDog。当然，你可能让一个 EOCPerson 的实例挡雨一个它的子类的实例；例如，EOCSmithPerson。这展示了一个继承层次的相等性中的共同问题。当你实现你的 isEqual: 方法时，你必须考虑这一点。最后，每个属性被检查相等性。如果它们中有任何一个不相等，则两个对象不想等；否者，它们就是相等的。

还有 hash 方法。回忆一下协议，相等对象必须返回同样的 hash，但是拥有相同 hash 的对象并不必须要相等。因此，如果你重写了 isEqual:，重写 hash 也是必要的。一个完美的可接受的 hash 方法如下：

	- (NSUInteger)hash {
    	return 1337;
	}

可是，如果你将这些对象放入容器中，这将会导致性能问题，因为 hash 被用于容器所用到的哈希表的索引。一个 set 的实现可能会根据哈希值将对象插入到不同的数组中。然后，当一个对象被加入到 set 时，符合它的哈希值的数组被遍历，以观察数组中是否有对象和其相等。如果，有相等的对象，则说明对象已经在 set 中。因此，如果你为每个对象返回相同的哈希值，钥匙你在 set 中添加1000000个对象，每次再添加就需要扫描这1000000个对象。

另外一中 hash 方法的实现可能为：

	- (NSUInteger)hash {
    	NSString *stringToHash =
        	[NSString stringWithFormat:@"%@:%@:%i",
        	    _firstName, _lastName, _age];
    	return [stringToHash hash];
	}

这一次， NSString 的 hash 方法的算法被用于返回一个新创建的字符串的 hash。这么做符合协议，因为两个 相等的 EOCPerson 对象将总是返回相同的 hash。可是，这种方法的不足在于它比单单返回一个值要慢很多，因为你有创建字符串的开销。在向一个集合加入对象时，这会引起性能问题，因为向集合加入对象时，需要计算它的 hash。

第三个创建 hash 的最终方法如下：

	- (NSUInteger)hash {
    	NSUInteger firstNameHash = [_firstName hash];
    	NSUInteger lastNameHash = [_lastName hash];
    	NSUInteger ageHash = _age;
    	return firstNameHash ^ lastNameHash ^ ageHash;
	}

这个方法在效率和至少取得一定范围内的 hash 之间取得了平衡。当然，使用这个算法，这会有一些冲突，但是至少 hash 函数的返回值不是唯一的。冲突频率和 hash 方法的计算密度之间平衡是一件你必须实验，以发现适合你的对象的算法。

### 类的特殊相等方法
除了之前描述的 NSString，其它提供了自有的相等方法的类包括 NSArray（isEqualToArray:）和 NSDictionary （isEqualToDictionary:），它们在被比较的对象不是数组或者字典时，分别都会抛出异常。Objective-C 并不在编译时强制类型检查，所以你可能很容易偶然传入错误类型的对象。因此，你需要确认你传入的对象确实是正确的类型。

你可能决定创建你自己的相等方法，如果相等性可能是经常检查的话；省略类型检查而获得得额外得速度是显著的。另外一个提供特别方法的理由是纯粹表面化的，你认为这样看起来更稳易读，就如同 NSString 的 isEqualToString: 方法的部分目的一样。使用这个方法的代码更易读，因为你不必费心考虑被比较的两个对象的类型。

如果，你创建了一个特殊方法，你必须重写 isEqual: 方法，并检查两个对象的类型。如果不是相同类型，通常实践是传给父类的实现。例如，类 EOCPerson 可以如下实现：

	- (BOOL)isEqualToPerson:(EOCPerson*)otherPerson {
    	if (self == object) return YES;

    	if (![_firstName isEqualToString:otherPerson.firstName])
        	return NO;
    	if (![_lastName isEqualToString:otherPerson.lastName])
    	    return NO;
	    if (_age != otherPerson.age)
        	return NO;
    	return YES;
	}

	- (BOOL)isEqual:(id)object {
    	if ([self class] == [object class]) {
    	    return [self isEqualToPerson:(EOCPerson*)object];
	    } else {
        	return [super isEqual:object];
    	}
	}

### 深度相等 VS 浅度相等
当你创建一个相等方法时，你需要决定是否检查整个对象或者仅仅检查几个变量。NSArray 检查两个数组是否包含相同数量的对象，如果是，遍历它们对每个成员调用 isEqual: 方法。如果所有的对象都相等，这两个数组就确实相等，这叫深度相等。

有时，可是，如果你知道几个数据可以决定相等性，那么不为相等性检查所有数据是可行的。

例如，使用 EOCPerson 类，如果实例来源于一个数据库，它们可能有另外一个附加的以数据库的主键作为唯一标示的属性：

	@property NSUInteger identifier;

在这样一个场景中，你可能决定只检查标示是否匹配。特别是属性被声明为外部只读，这样你可以确定两个有相同标示的对象，它们确实代表相同的对象，它们是相等。由于来源于相同的数据源，当你可以断言标示匹配，剩余的数据也匹配时，这就不用检查 EOCPerson 对象包含的每一位数据。

是否在相等方法中检查所有的变量完全取决于各个对象。只有你能知道你的对象实例相等到底意味着什么。

### 容器中可变类的相等性
一个需要考虑重要场景是可变对象被放入容器中。一旦你向集合添加对象，它的哈希值不应该变化。之前，我解释过对象更具它们的哈希值被分入不同的区域。如果它们的哈希值在被分区时变化了，对象就将在错误的区中。为了解决这个问题，你可以保证只要对象在集合中，它们的哈希值不却取决于对象的可变部分，或者简单地就不可变。在条目 18
中，我会解释为什么你应该让对象不可变。这是那个理由的一个绝好例子。

你可以看到测试 NSMutableSet 和 NSMutableArray 行为中的问题。从将一个对象加入 set 开始：

	NSMutableSet *set = [NSMutableSet new];

	NSMutableArray *arrayA = [@[@1, @2] mutableCopy];
	[set addObject:arrayA];
	NSLog(@"set = %@", set);

Set 包含一个对象：两个对象的数组。现在添加一个包含有相同顺序的相同对象的数组，这样新的数组和已经存在于 set 中的数组相等：

	NSMutableArray *arrayB = [@[@1, @2] mutableCopy];
	[set addObject:arrayB];
	NSLog(@"set = %@", set);
	// Output: set = {((1,2))}

Set 仍然包含一个对象，因为要添加的对象等于已经存在于 set 中的对象。现在我们向 set 中添加一个和已在 set 中的数组不相等的数组：

	NSMutableArray *arrayC = [@[@1] mutableCopy];
	[set addObject:arrayC];
	NSLog(@"set = %@", set);
	// Output: set = {((1),(1,2))}

如期望的，set 现在包含两个数组：原来的那个和新添加的，因为 arrayC 不等于已存在于 set 中的数组。最终，我改变 arrayC 为等于另一个存在于 set 的数组：

	[arrayC addObject:@2];
	NSLog(@"set = %@", set);
	// Output: set = {((1,2),(1,2))}

嗯，哦，亲爱的，现在 set 中的两个数组彼此相等了！一个 set 不允许这种情况，但是它已不能维持它的语义了，因为你已改变了已存在于 set 中的一个对象。更为尴尬的时如果这个 set 之后又被拷贝了：

	NSSet *setB = [set copy];
	NSLog(@"setB = %@", setB);
	// Output: setB = {((1,2))}

拷贝出来的 set 只有一个对象在里面，就如同 set 被以空 set 创建，再一个一个往里添加被拷贝 set 中的对象。这可能是你期望的，也可能不是。你可能需要逐字拷贝原有对象，即使又错误存在。或者，你可能希望它如同它之前做得那样。两者都将是有效的复制算法，进一步说明了这一点，集合存在错误时，一切在其上的操作都成为了赌博。

这个故事的寓意是当你改变了集合中的数据，那就要对将发生的事情保持警惕。这不是说你不应该这么做，而是你应该潜在的问题和相应的代码保持警惕。

### 要点回顾
* 为你将要做相等性检查的对象提供 isEqual: 和 hash 方法。

* 相等对象必须有相同的哈希值，但是有相同哈希的对象并必须要相等。

* 确定哪些属性是测试相等性所必须的，而不是简单粗暴地检查所有属性。

* 编写的 hash 方法必须快速，同时提供一个合理的低冲突概率。


## 条目 9：使用类簇模式隐藏实现细节

## 条目 10：通过关联对象（Associated Object）将用户数据挂载在已有类上
有时，你要将信息与对象关联。正常情况下，你会通过继承对象的类，在子类中添加信息。可是，你并不总能这么做，因为为你创建类的实例的方法并不一定支持创建你的类的实例。这就是强大的 Objective-C 的一个名为关联（Associated Object）的功能派上用场的地方。

对象被与其它对象关联起来，通过键标示这些相关对象。它们也可以被指定不同的存储策略，使得被存储的值应用相应的内存管理语义。存储策略由枚举类型 objc_AssociationPolicy 定义，它包含的值如表 2.1 所示一一对应了 @property 的参数（见条目 6 以获取更多关于属性的信息）。

![alt Associated-Object](/images/blog/EffectiveObjC/Associated-Object.png "Associated Object")

**表 2.1**对象关联类型

关联的管理使用如下方法实施：

* 以给定的键和存储策略建立一个 value 的对象关联。

	void objc_setAssociatedObject(id object, void *key, id value, objc_AssociationPolicy policy)

* 返回 object 上对应于给定键的关联。

	id objc_getAssociatedObject(id object, void *key)

* 去除 object 上所有的关联。

	void objc_removeAssociatedObjects(id object)

访问关联对象功能上类似于想像 object 是一个 NSDictionary，并调用 [object setObject:value forKey:key] 和 [object objectForKey:key]。但一个重要的不同要注意， 键被完全当作不透明的指针。由于在字典中，如键的 isEqual: 方法返回 YES，则键相等，而关联对象的键必须是完全相同的指针，关联对象才相等。鉴于这个原因，通常使用静态全局变量作为广联对象的键。

### 一个使用关联对象的例子
在 iOS 开发中，类 UIView 是很常用的，它提供了一个向用户显示警告框的标准视图。存在一个委托协议用于处理用户点击按钮关闭它的事件；可是，使用委托分割了创建警告框和处理点击的代码。着使得代码读起来有点麻烦，因为代码被分割在两个地方。这有一个通常如何使用 UIAlertView 的例子：

	- (void)askUserAQuestion {
    	UIAlertView *alert = [[UIAlertView alloc]
        	                     initWithTitle:@"Question"
            	                   message:@"What do you want to do?"
                   	              delegate:self
                	        cancelButtonTitle:@"Cancel"
                   	     otherButtonTitles:@"Continue", nil];
	      [alert show];
	}

	// UIAlertViewDelegate protocol method
	- (void)alertView:(UIAlertView *)alertView
    	    clickedButtonAtIndex:(NSInteger)buttonIndex
	{
    	if (buttonIndex == 0) {
        	[self doCancel];
	    } else {
    	    [self doContinue];
	    }
	}

如果你试图在同一个类中展示多余一个警告框，这个模式甚至会更复杂，因为你必须检查传入 delegate 方法的 alertView 参数，并根据不同 alertView 切换。如果当每个按钮被点击的逻辑在警告框被创建时就能被确定，那就会简单很多。解决方法是在创建警告框时，为它设置一个块（block），然后在 delegate 方法运行时读出这个块。它实现起来如下：

	#import <objc/runtime.h>

	static void *EOCMyAlertViewKey = "EOCMyAlertViewKey";

	- (void)askUserAQuestion {
    	UIAlertView *alert = [[UIAlertView alloc]
        	                     initWithTitle:@"Question"
            	                   message:@"What do you want to do?"
                	                  delegate:self
                   	     cancelButtonTitle:@"Cancel"
                   	     otherButtonTitles:@"Continue", nil];

      	void (^block)(NSInteger) = ^(NSInteger buttonIndex){
        	if (buttonIndex == 0) {
            	[self doCancel];
	        } else {
    	        [self doContinue];
        	}
	    };

     	objc_setAssociatedObject(alert,
        	                     EOCMyAlertViewKey,
            	                 block,
            	                 OBJC_ASSOCIATION_COPY);

	    [alert show];
	}

	// UIAlertViewDelegate protocol method
	- (void)alertView:(UIAlertView*)alertView
    	    clickedButtonAtIndex:(NSInteger)buttonIndex
	{
    	void (^block)(NSInteger) =
        	objc_getAssociatedObject(alertView, EOCMyAlertViewKey);
	    block(buttonIndex);
	}

通过这个方法，创建警告框的代码和处理结果的代码都在同一个地方，这使得代码比之前更可读一些，因为你不必在两块代码之间摆动以理解为何警告框试图被使用了。但你仍然要对这个方法小心翼翼，因为块的使用中很容易产生循环引用。见条目 40 以获得更多关于这个问题的信息。

如你所见，这个方法是很强大的，当时它只应该在其它方法都不凑效时被使用。广泛使用这个方法可能会很快失控，导致调试困难。因为在关联对象之间不存在正式关系定义，循环引用变得很难理解，内存管理语义时在关联时才被定义而不是在定义接口时被定义。所以使用这个方法时要小心谨慎，不要只是因为你能使用就是去使用它。一个完成 UIAlertView 的替代方法时继承它，并给它添加一个用于存储块（block）的属性。如果 alert 视图被使用不只一次，我更建议使用这个方法，而不是关联对象。

### 要点回顾
* 关联对象提供了一种将两个对象联系在一起的方法。
* 关联对象的内存管理语义可以被定义为类似拥有或不拥有关系。
* 关联对象只应该在其它方法都不能用时才被使用，因为它们很容易导致难以调试的 bug。

## 条目 11：理解 objc_msgSend 的角色
在 Objective－C 中，调用对象的方法是最常见事情之一。在 Objective-C 术语中，这被称为发送消息。消息有名字，或成为选择器（selector），有参数，可能会有返回值。

因为 Objective-C 是 C 的超集，所以从理解 C 中函数调用开始会是个好注意。C 的函数调用是静态绑定的，这意味着在编译时函数的调用就被知晓了。例如，考虑如下例子：

	#import <stdio.h>

	void printHello() {
		printf("Hello, world!\n");
   	}
   	
   void printGoodbye() {
    	printf("Goodbye, world!\n");
	}

	void doTheThing(int type) {
    	if (type == 0) {
        	printHello();
	    } else {
    	    printGoodbye();
	    }
    	return 0;
	}

忽略内联，当这段代码被编译时，printHello 和 printGoodbye 是被知道的，编译器可发送指令直接调用函数。函数的地址被高效地硬编码入指令。现在考虑一下这么写的代码：

	#import <stdio.h>

	void printHello() {
   		printf("Hello, world!\n");
	}
	void printGoodbye() {
    	printf("Goodbye, world!\n");
	}

	void doTheThing(int type) {
    	void (*fnc)();
    	if (type == 0) {
        	fnc = printHello;
	    } else {
    	    fnc = printGoodbye;
	    }
    	fnc();
    	return 0;
	}

这里，用到了动态绑定，因为被调用的函数到运行时才知道。编译器发出的指令将不同与第一个例子中，在 if 和 else 分支中都有函数调用。在第二个例子中，只有一个单一的函数调用，但却有读取被调用的那个函数的地址的消耗，地址并没有被硬编码。

在 Objective-C 中动态绑定是这样机制，当消息被传送给一个对象时方法被调用。所有方法其实底层都是纯 C 函数，但是消息到来时哪一个函数被调用完全决定于运行时，并且甚至可以在应用的整个运行过程中被修改，这使得 Objective-C 成为纯粹动态语言。向对象发送消息看起来如下：

	id returnValue = [someObject messageName:parameter];

在这个例子中，someObject 被作为接收者，messageName 是选择器。选择器和参数合在一起被成为消息。编译器看到这条消息时，它将消息转为标准 C 函数，这个函数是消息发送的核心函数, objc_msgSend,它有如下原型：

	void objc_msgSend(id self, SEL cmd, ...)

这是一个可变参数函数，可以有两个或更多参数。第一个参数是接收者，第二个参数是选择器（SEL 是选择器的类型），剩余的参数为消息参数，按消息参数的顺序。选择器是被视为方法的名字。选择器这个词常常页被用在方法这个词试用的地方。前面的例子中的消息可以转化为如下：

	id returnValue = objc_msgSend(someObject,
                      	          @selector(messageName:),
                                 parameter);

objc_msgSend 函数能否调用正确的方法，取决于接受者的类型和选择器。为了能够正确调用，行数遍历接收者的类所实现的方法列表，如果找到一个方法匹配选择器的名字，就跳往这个方法实现。如果没有发现匹配的，则跳往继承结构向上层层追溯知道找到匹配的方法。如果没有匹配的方法被找到，消息转发接入。为了获取更多解释，查看条目 12。

看起来无论何时方法被调用，都要做很的的查找工作。幸运的是，objc_msgSend 在一个快速图中缓存了结果，每个类都有对应的图，这样对于相同类以后的消息，选择器会很快被执行。即使这个快速方法比静态绑定函数调用要慢，但是一旦选择器被缓存了，这个差距不会太大；实际上，在应用中消息分发并不是瓶颈。如果它是瓶颈，你可以写一个 C 函数，通过 Objective-C 对象的各种状态，调用它。

前面的描述只是针对特定的消息。Objective-C 运行时环境提供了更多的函数用于处理一些特定的边缘情况：

* objc_msgSend_stret
  发送返回结构体的消息。objc_msgSend 这个函数只能处理返回可存入 CPU 寄存器的特定类型的消息。如果返回类型不符合，例如，如果返回一个结构体，另一个函数江北调用以执行分发。在这个例子中，另一个函数 objc_msgSend_stret 被调用，通过栈分配变量来处理返回结构体的情况。
  
* objc_msgSend_fpret
  发送返回浮点数的消息。在函数调用中处理浮点数寄存器是需要一些特别的结构的，这意味着标准的 objc_msgSend 不够好。objc_msgSend_fpret 这个函数存在就是为了处理例如 x86 这样的有些不同的构架突然出现。

* objc_msgSendSuper
  向父类发送消息，例如 [super message:parameter]。这也有 objc_msgSend_stret 和 objc_msgSend_fpret 向父类发送消息的对等函数：objc_msgSendSuper_stret 和 objc_msgSendSuper_fpret。

我之前提过 objc_msgSend 这一系列函数一旦找到正确的方法实现便会跳往这个实现。每个 Objective-C 对象的方法可以被视为一个简单的 C 函数，它的原型形式如下：

	<return_type> Class_selector(id self, SEL _cmd, ...)

函数的名字并不真正如此，但是这是为了展示这一点：它是类和选择器的联合。这样的函数指针被存储在每个类的表中，以键对应选择器名字。这就是 objc_msgSend 系列方法查找实现所要跳入的地方。注意到这个原很型类似于 objec_msgSend 函数本身。这不是巧合。这是的跳入相应的函数更简单，也可更好试用尾部的调用优化。

尾部调用优化发生在一个函数最后一件事为调用另外一个函数时。编译器不再推入一个新的栈帧，而是发出跳到下一个函数的指令。这只能在一个函数最后做的事情是调用另外一个函数，并且不需要用到返回值时。对于 objc_msgSend 使用这项优化是很重要的，因为没有它，栈轨迹显示在每个 Objective-C 函数前都会有 objc_msgSend。栈溢出将过早发生。

实际上，Objective-C 中，你无需担忧所有的这些，但是理解这些隐藏的基础性的知识是有利的。如果你理解消息发送时发生了什么，你便能体会你的代码是怎么执行的，并了解为何在调试时你总会在回溯中看到 objc_msgSend。

### 要点回顾
* 一条消息由接收者，选择器，参数构成。调用消息是在对象中调用方法的同义词。
* 当被调用时，所有消息将通过动态消息分发系统，这样消息的实现部分会被查找到并被运行。


## 条目 12：理解消息转发

条目 11 解释了理解消息如何被发向对象是重要的。条目 12 将探索了解当一条消息被发向一个无法理解它的对象时发生了什么为何是重要的。

一个类只能理解那些它通过编码实现了方法的消息。但是向一个无法理解此消息的对象发送消息，不是一个编译时错误，因为方法是再运行时被挂到类上的，所以编译器无法知道一个方法实现运行时是否会存在。当对象收到一个它无法理解的方法时，它会进行消息转发。消息转被设计出来是为了让开发者可以告知消息如何处理未知消息。

即使没有意识到消息转发存在，你也很可能已经遇到了被消息转发转发的消息了。你遇到如下面控制台中显示的消息，这都是因为你向一个对象发送了它无法理解的消息，而被转发到缺省的 NSObject 实现。

	-[__NSCFNumber lowercaseString]: unrecognized selector sent to instance 0x87
	*** Terminating app due to uncaught exception 'NSInvalidArgumentException', 
	reason: '-[__NSCFNumber lowercaseString]: unrecognized selector sent to instance 0x87'

这是一个被 NSObject 的 doesNotRecognizeSelector 方法抛出的异常，告知你消息的接收器是 __NSCFNumber 类型它无法理解选择器 lowercaseString。在这个例子中这并不让你奇怪，因为 NSNumber（_NSCFNumber 是在 toll－free bridging 中使用的内部类，当你申请了一个 NSNumber，内部会创建 _NSCFNumber,见 条目 49）。在这个例子中，转发路径终结于应用崩溃，但是你在你自己的类的转发路径中，有勾子可以使你可以执行任何你想要的逻辑，而代替崩溃。

转发路径被分叉为两条路。第一条给接收器所属的类一个机会可以动态的为未知选择器添加一个方法。这叫做动态方法解析。第二条是全转发机制。如果运行时走了这条路，说明接收器已无机会回应某个选择器。所以，让接收器自己处理无法处理的方法调用，它会分两步来做。首先，它询问是否有其它对象应该替代它接收这个消息。如果有，运行时转发消息，所有事情都如常运行。如果没有替代的接收器，全转发机制将产生作用， NSInvocation 对象包装了所有有关细节：消息现在无法被处理，再给接收器最后一个处理的机会。

### 动态方法解析

当方法被传给一个无法理解它的对象时，第一个被调用的方法是一个类方法：
	
	+ (BOOL)resolveInstanceMethod:(SEL)selector

这个方法的传入参数为对象找不到的那个选择器，返回一个布尔变量以说明是否有一个实例方法被添加在这个类，使其可以处理这个选择器。因此，类被给予了的第二个机会通过转发机制在运行之前，添加一个实现。一个类似的方法，名为 resolveClassMethod：，在未被类实现的方法是类方法而不是实例方法时会被调用。

使用此方案要求的方法实现已经存在，只待动态的插入类。这种方法通常使用 @dynamic 属性实现（见条目 6），如 CoreData 中访问 NSManagedObjects 的属性时发生的那样，因为被要求实现的方法在编译时是可以知的。

例如，一个为了使用 @dynamic 属性而实现的 resolveInstanceMethod: 看起来可能如下：

	id autoDictionaryGetter(id self, SEL _cmd);
	void autoDictionarySetter(id self, SEL _cmd, id value);

	+ (BOOL)resolveInstanceMethod:(SEL)selector {
    	NSString *selectorString = NSStringFromSelector(selector);
    	if ( /* selector is from a @dynamic property */ ) {
        	if ([selectorString hasPrefix:@"set"]) {
            	class_addMethod(self,
                	            selector,
                   		        (IMP)autoDictionarySetter,
                   	         	"v@:@");
	        } else {
    	        class_addMethod(self,
        	                    selector,
            	                (IMP)autoDictionaryGetter,
	              	            "@@:");
	        }
        	return YES;
    	}
	    return [super resolveInstanceMethod:selector];
	}

选择器被以字符串类型获取，之后检查它是否是一个赋值器。如果它以 set 为前缀，便被认为是一个赋值器；否则，它被认为是一个取值器。在每种情况下，一个方法都被添加到类中，为给定的选择器指定一个纯 C 函数的实现。在这个 C 函数中，将有代码可以执行将可以被类使用的任何一种数据结构存入属性数据。例如，在 CoreData 的例子中，这些方法将和数据库后端沟通以根据需要提取或则更新数据。

### 替代接收器
第二个处理未知选择器的尝试是询问接收器是否有替代接受器可以处理消息。处理这个的方法是：

	- (id)forwardingTargetForSelector:(SEL)selector

未知选择器被传入，接收器被期望放回可做代替的对象，如果无法找到替代者则返回 nil。这个方法可以被用于通过组合也可提供类似多继承的某些便利。一个对象可以在内部拥有这个方法提供的一组其它对象，用于处理选择器，就好像它自己能够处理一样。

注意这种转发方法不能操作消息。如果消息需要在被发送至替代者之前被修改，则需使用到全转发机制。

### 全转发机制
如果转发算法已经到达了这一步拉了，那应用全转发机制便是唯一可以做的事情了。全转发通过创建一个 NSInvocation 对象开始，该对象包装所有的关于待处理消息的细节。这个对象包含了选择器，目标（target），和参数（argument）。一个 NSInvocation 对象可以被调用，这将引起消息分发系统进入工作，并将消息风发给它的目标。

被调用以尝试转发的方法是：

- (void)forwardInvocation:(NSInvocation*)invocation

一个简单实现将改变 invocation 的目标，并调用 invocation。这会等价于使用替代接收器方法，像这样的简单实现很少被用到。一个更有用的实现是在调用消息之前，通过某种方法改变消息，例如添加另外一个参数或者改变选择器。

这个方法的一个实现应该总是调用它的父类的实现以处理它无法处理的消息。这意味着继承结构中所有的父类都被给予一次处理这个调用的机会，NSObject 的实现将被调用。这将会导致 doesNotRecognizedSelector: 被调用以抛出一个无法处理选择器的异常。

### 全图
转发被处理过程可以被如图 2.2 的流程图所描述。

![alt Message-Forwarding](/images/blog/EffectiveObjC/Message-Forwarding.png "Message-Forwarding")

每一步，接收器都有机会处理消息。每一步都比它之前的一步消耗更大。最佳场景是方法在第一步就被解析了，因为被解析的方法最终会被运行时缓存，以至于第一次之后在相同类的实例上调用相同选择器的转发无需深入。在第二步，转发一条消息给另外一个接收器是第三步在替代接收器可被找到的情况下的一种简单优化。在这种情况下，调用唯一需要改变的事情是目标（target），相对于最后一步中一个完整的 NSInvocation 需要被创建和处理，改变目标是非常简单的。

### 动态方法解析的完整实例

为了展示转发的作用，下面的例子显示了使用动态方法解析提供一个 @dynamic 属性。考虑到一个对象。考虑一个对象允许你储存任何对象，就像一个字典，但是是通过属性访问。这个类的实现是你可以添加一个属性定义，并声明为 @dynamic，类将魔法般处理值的存储和获取。这看起来很棒，不是吗？

类的接口如此：
	
	＃import <Foundation/Foundation.h>
	
	@interface EOCAutoDictionary : NSObject
	@property (nonatomic, strong) NSString *string;
	@property (nonatomic, strong) NSNumber *number;
	@property (nonatomic, strong) NSData *date;
	@property (nonatomic, strong) id opaqueObject;
	@end
	
对这个例子来说，有何种属性并不重要。实际上，已展示的各种类型仅仅意在表现这个功能的强大。每个属性的值在内部实际上还是存储在字典中，所以类的实现的开头会如下，包括以 @dynamic 声明属性，以使得实例变量和访问器不会被自动生成：

	#import "EOCAutoDictionary.h"
	#import <objc/runtime.h>
	
	@interface EOCAutoDictionary ()
	@porperty (nonatomic, strong) NSMutableDictionary *backingStore;
	@end
	
	@implementation EOCAutoDictionary
	@dynmic string, number, date, opaqueObject;
	
	- (id)init {
		if ((self = [super init]) {
			_backStore = [NSMutableDictionary new];
		}
		return self;
	}
	
然后到了有趣的部分了： resolveInstanceMethod: 实现：

	+ (BOOL)resolveInstanceMethod:(SEL)selector {
    	NSString *selectorString = NSStringFromSelector(selector);
    	if ( /* selector is from a @dynamic property */ ) {
        	if ([selectorString hasPrefix:@"set"]) {
            	class_addMethod(self,
                	            selector,
                   		        (IMP)autoDictionarySetter,
                   	         	"v@:@");
	        } else {
    	        class_addMethod(self,
        	                    selector,
            	                (IMP)autoDictionaryGetter,
	              	            "@@:");
	        }
        	return YES;
    	}
	    return [super resolveInstanceMethod:selector];
	}
	
	@end
	
第一次遇到 EOCAutoDictionary 的一个属性调用时，运行时会找不到对应的选择器，因为它们没有被直接实现，也没有被自动生成。例如，如果 opaqueObject 属性被赋值，上面的方法将被以选择器 setOpaqueObject: 为参数调用。类似地，如果属性被读取，它将被以选择器 opaqueObject: 为参数调用。方法通过检查是否有 set 前缀，侦测 set 和 get 选择器之间地不同。在每种情况下，一个方法被添加到类中，以为给定的选择器指定一个函数调用，这里按情况选择  autoDictionarySetter 或者 autoDictionaryGetter。这使用到了运行时方法 class_addMethod，它为选择器在类中，动态添加一个方法，该方法以函数指针给出。class_addMethod 函数的最后一个参数是实现的编码类型。编码类型由字符串表示返回类型，以及随后的函数的参数。

取值器的实现如下：

	id autoDictionaryGetter(id self, SEL _cmd) {
		// Get the backing store form the object
		EOCAutoDictionary *typedSelf = (EOCAutoDictionary *)self;
		NSMutableDictionary *backingStore = typedSelf.backingStore;
		
		// The key is simply the selector name
		NSString *key = NSStringFromSelector(_cmd);
		
		// Return the Value
		return [backingStore objectForKey:Key];
	}
	
最后，赋值器的实现如下：

	void autoDictionarySetter(id self, SEL _cmd, id value) {
		// Get the backingStore from the object
		EOCAutoDictionary *typedSelf = (EOCAutoDictionary *)self;
		NSMutableDictionary *backingStore = typedSelf.backingStore;
		
		/** The selector will be for example, "setOpaqueObject:"
		 *  We need to remove the "set", ":" and lowercase the first
		 *  letter of the remainder
		 */
		NSString *selectorString = NSStringFromSelector(_cmd);
		NSMutableString *key = [selectorString mutableCopy];
		
		// Remove the ':' at the end
		[key deleteCharactersInRange:NSMakeRange(key.length - 1, 1)]; 
		
		// Remove the 'set' prefix
		[key deleteCharactersInRange:NSMakeRange(0, 3)];
		
		// Lowercase the first character
		NSString *lowercaseFirstChar = [[key substringToIndex:1] lowercaseString];
		[key replaceCharactersnRange:NSMakeRange(0, 1) withString:lowercaseFirstChar];
		
		if (value) {
			[backingStore setObject:value forKey:key];
		} else {
			[backingStore removeObjectForKey:key];
		}
		
	}
	
然后，使用 EOCAutoDictionary 就是件很简单的事了：

	EOCAutoDictionary *dict = [EOCAutoDictionary new];
	dict.date = [NSdate dateWithTimeIntervalSince1970:475372800];
	NSLog(@"dict.date = %@", dict.date);
	// Output dict.date = 1985-01-24 00:00:00 +0000
	
字典上的其它属性可以如同 date 属性一样被访问，新的属性可以通过添加 @property 定义和声明其为 @dynamic 引入。iOS 上的 CoreAnimation 库中的 CALayer 采用了类似的方法。这个方法允许 CALayer 成为一个键值编码兼容的容器类，意味着它可以存储键值对。CALayer 使用这种能力以允许添加客户自定义的动画属性，据此属性值的存储直接由基类完成，而属性定义可以在子类中添加。

### 要点回顾
* 消息转发时一个对象在被发现无法回应一个选择器时所需经历的过程
* 动态方法解析被用于在运行时向类添加方法
* 对象可以声明其它对象来处理它所不能处理的特定选择器
* 完全转发只发生在上述处理选择器的方法都没使用的情况下

## 条目 13：考虑使用 Swizzling 方法调试黑箱方法

## 条目 14：理解对象类是什么