---
layout: post
title: Swift中的设计模式 
description: “Swift 的编程范式” 之八：这篇短文中，我们回顾了设计模式的概念，并讨论了在现代编程语言背景下，设计模式的一些变化。然后使用Swift语言演示了5个设计模式和新的语言特性之间的关系：高阶函数和命令模式，一等函数和策略模式，柯里化函数和抽象工厂模式，扩展和适配器模式，运算符重载和解释器模式。
category: blog
---

## 设计模式

设计模式（Design Pattern）是对软件设计中普遍存在的各种问题，所提出的解决方案。这个术语是由埃里希·伽玛等人（Erich Gamma，Richard Helm，Ralph Johnson和John Vlissides 这四人。也被称为：Gang of Four，GOF，四人帮）在1990年代从建筑设计领域引入到计算机科学的。 设计模式并不能直接用于完成代码的编写，而是描述在各种不同情况下，要怎么解决问题的一种方案。

以上描述摘自维基百科。设计模式出现于1990年代，那时主要的编程语言是Java和C++这样的面向对象编程语言。随着计算机语言的发展，越来越多的现代语言的出现和流行，已经改变了设计模式出现时的计算机语言背景。虽然，现在Java和C++依然流行，但Python，Scala，Groovy，Swift等现代语言使用者越来越多。函数式编程语言（Lisp，Scheme，Haskell等）也越来越来受到重视。程序员的视野大大拓展了，手中可使用的工具也更丰富了。

所以现在有一部分人质疑设计模式已经与时代发展脱节，认为设计模式是过时的产物。设计模式解决问题的方式，增加了不必要的复杂度。在一些现代语言中，设计模式解决的问题可以更好地被其他方式处理，有些问题甚至可能根本不存在。

这种质疑一定程度上是正确的。因为，我们现在拥有了更高级的语言工具。语言的选择是很重要的，因为语言的选择会影响我们看问题的方式。在使用过程式语言时，你可能为了方便读取数据，而“制造”某种类似于类（class）的数据结构，你可能会称这种设计模式为“封装“。但在面向对象编程语言中，你不会认为这是一个需要设计模式来解决的问题，因为语言本身已经为你解决了。在使用面向对象语言时，为了使操作可以在对象间传递，我们使用一个被称为命令模式（Command）的设计模式。但是，在拥有高阶函数的编程语言中，你可能没有意识到这是个问题。因为，你很自然地就将函数（即操作）作为参数传入到另一个函数中，而这就是函数式编程语言中高阶函数的概念。

从这点来看，随着我们所使用的编程语言的演化，我们遇到的问题也确实一直在改变。GOF提出的23种设计模式也许部分过时了。但我们遇到的问题并不会消失，设计模式的概念将一直存在：提炼普遍存在的问题，提出解决方案。这其实是一个抽象过程。现在已有的编程语言都存在表达的局限，即对某类问题抽象层次过低。所以，我们在使用任何编程语言时，都还是会遇到一些普遍存在的却没有被语言本身很好解决的问题。这时，我们就会使用到“设计模式”，即人们总结出来的解决方案：遇到问题A，用方案A；遇到问题B，用方案B。只不过问题会一直变化。现在我们可以不再使用那23个模式来解决问题了，但是我们仍然需要总结出其它模式来解决新的问题。这种情况一直会持续到我们拥有“完美语言”的那一天。但现在看起来，这一天还没有到来的迹象。

### Swift中的设计模式

Swift是一门多范式编程语言。除了支持面向对象编程范式之外，还支持函数式编程范式，和泛型编程。这使得Swift可以使用函数式编程的某些优秀工具解决我们在面向对象编程中遇到的困难。作为一门崭新的语言（2014年发布），Swift也同时吸收了很多其他语言的优秀特性。这使得与GOF首次提出23个设计模式时相比，我们现在看待Swift中设计模式相关问题的看法肯定已大大不同了。毕竟GOF提出设计模式主要是针对Java和C++这种相对而言更传统的编程语言。在Swift中，一些模式已经被语言特性所吸收，你在使用Swift甚至察觉不出这类问题的存在；一些问题仍然存在，我们仍然需要某种设计模式，但实现起来会更为简便；当然，仍然会存在一些问题，Swift也没有解决。

另外一个有趣的相反的问题是：Swift是否引起了一些新问题，需要一些新的设计模式来解决呢？相对于一些实验性编程语言（如Lisp，Haskell）而言，作为一门工业语言，Swift的语言设计其实是保守的，吸收的大都是其他语言成熟的优秀特性。本质上，Swift仍然主要还是一门面向对象编程语言。函数式编程，泛型编程等新的语言特性的引入，都只是一种改进，还谈不上是革命性的变化。这使得Swift虽然显得保守，但也风险可控，伴随剧烈革命而引入新的问题的可能性被降低了。当然，最终结论仍然需要大规模的业界应用实践之后才能得出。

这篇文章主要会涉及Swift改善的那一部分。具体而言，是指Swift消除了哪些设计模式，使哪些设计模式的实现简化了。

## 函数式编程

Swift支持函数式编程范式。程序员可以使用Swift写出函数式风格的代码。函数式编程是一种以数学函数为程序语言建模的核心的编程范式。在函数式编程中，函数是核心概念，是“头等公民”，函数被赋予了更多职责，拥有更多灵活的使用方式。这一章可以看到使用函数式编程范式，可以消除一些面向对象编程中使用到的设计模式。

### 高阶函数

高阶函数，指可以将其他函数作为参数或者返回结果的函数。由于高阶函数，我们发现GOF设计模式中命令模式（Command）在Swift中消失了。

命令模式使用对象封装一系列操作（命令），使得操作可以重复使用，也易于在对象间传递。由于Swift仍然主要是一门面向对象编程语言，我们仍然可以使用Swift实现一个经典的命令模式。实现命令模式的目的只是和之后使用高阶函数的方案对比：

```
	// Receiver
	class Light {

  		func turnOn() {
    		println("The light is on")
  		}
	
  		func turnOff() {
    		println("The light is off")
  		}
	}

	// Abstract Command
	protocol Command {
  		func execute()
	}

	// Concrete Command
	class FlipUpCommand: Command {
  		private let light: Light

	  	init(light: Light) {
    		self.light = light
  		}

  		func execute() {
    		light.turnOn()
  		}
	}

	class FlipDownCommand: Command {
  		private let light: Light

  		init(light: Light) {
    		self.light = light
  		}

  		func execute() {
    		light.turnOff()
  		}
	}
```

以上例程中，灯（Light）是命令（Command）的操作对象（Receiver）。我们定义了命令的协议，同时我们实现两个具体的命令操作：FlipUpCommand和FlipDownCommand。它们分别使灯亮，和使灯灭。

```
	// Invoker
	class LightSwitch {

  		var queue: [Command] = []

  		func addCommand(command: Command) {
    		queue.append(command)
  		}

  		func execute() {
    		for command in queue {
      			command.execute()
    		}
  		}
	}

	// Client
	class Client {

  		static func pressSwitch() {
    		let lamp = Light()
    		let flipUpCommand = FlipUpCommand(light: lamp)
    		let flipDownCommand = FlipDownCommand(light: lamp)

    		let switcher = LightSwitch()
    		switcher.addCommand(flipUpCommand)
    		switcher.addCommand(flipDownCommand)
    		switcher.addCommand(flipUpCommand)
    		switcher.addCommand(flipDownCommand)

    		switcher.execute()
  		}
	}

	// Use
	Client.pressSwitch()
```

这段则代码显示了如何使用命令模式。

在函数式编程中，由于存在高阶函数。我们可以直接将一个函数作为参数传给另外一个函数。所以，使用类包裹函数在对象间传递这件事情就显得多余了。以下代码显示如何使用高阶函数达到命令模式相同的效果：

```
	// Invoker
	class LightSwitchFP {

  		var queue: Array<(Light) -> ()> = []

  		func addCommand(command: (Light) -> ()) {
    		queue.append(command)
  		}

  		func execute(light: Light) {
    		for command in queue {
      			command(light)
    		}
  		}
	}

	// Client
	class ClientFP {

  		static func pressSwitch() {
    		let lamp = Light()
    		let flipUp = { (light: Light) -> () in light.turnOn() }
    		let flipDown = { (light: Light) -> () in light.turnOff() }

 	   		let lightSwitchFP = LightSwitchFP()
    		lightSwitchFP.addCommand(flipUp)
    		lightSwitchFP.addCommand(flipDown)
    		lightSwitchFP.addCommand(flipUp)
    		lightSwitchFP.addCommand(flipDown)

    		lightSwitchFP.execute(lamp)
  		}
	}

	// Use
	ClientFP.pressSwitch()
```

使用高阶函数的版本中，负责集中调度命令的LightSwitchFP类有一个接受命令的函数addCommand。由于Swift支持高阶函数，这个函数无需接受一个携带命令函数的Command对象，而是直接接受表示命令的函数。这样更为直接自然。所以，命令模式在Swift这样拥有高阶函数的编程语言中，就显得多余了。

### 一等函数

一等函数，进一步扩展了函数的使用范围，使得函数成为语言中的“头等公民”。这意味函数可在任何其他语言构件（比如变量）出现的地方出现。可以说，一等函数是更严格的高阶函数。

策略模式（Strategy）定义了一系列算法，将每个算法封装起来，并且使它们之间可以互相替换。此模式让算法的变化独立于使用算法的客户。

下面，我们使用Swift讨论一下一等函数对策略模式的影响。我们先用Swift实现传统的策略模式：

```
	protocol Strategy {
  		func compute(first: Int, second: Int) -> Int
	}

	class Add: Strategy {
  		func compute(first: Int, second: Int) -> Int {
    		return first + second
  		}
	}

	class Multiply: Strategy {
  		func compute(first: Int, second: Int) -> Int {
    		return first * second
  		}
	}

	class Context {
  		let strategy: Strategy

  		init(strategy: Strategy) {
    		self.strategy = strategy
  		}

  		func use(first: Int, second: Int) {
    		strategy.compute(first, second: second)
  		}
	}

	let context = Context(strategy: Add())
	context.use(1, second: 2)
```

类似于命令模式，策略模式中的策略对象主要用于封装操作（函数），不同的是策略模式中的策略对象封装的是不同的算法。这些算法实现了相同的接口，在这个例子中，接口是用Strategy协议表示的。我们使用两个实现了Strategy协议的具体类：`Add`和`Multiply`分别封装两个简单的算法。Context对象，用于对算法进行配置选择，它有一个Strategy类型的实例变量：strategy。通过配置Context的strategy具体类型，可以使用不同的算法。

然后我们再看看如果存在一等函数，策略模式是否可以得到化简：

```
	let add = { (first: Int, second: Int) -> Int in
  		return first + second
	}

	let multiply = { (first: Int, second: Int) -> Int in
  		return first * second
	}

	class ContextFP {

  		let strategy: (Int, Int) -> Int

  		init(strategy: (Int, Int) -> Int) {
    		self.strategy = strategy
  		}

  		func use(first: Int, second: Int) {
    		strategy(first, second)
  		}
	}

	let fpContext = FPContext(strategy: multiply)
	fpContext.use(1, second:2)
```

由于Swift的函数都是一等函数，使得我们可以把函数作为参数传给另外一个函数。这无需在使用对象来封装算法。函数可以成为封装算法的载体，这样更为直接自然。例子中，ContextFP的构造器的传参就是函数类型。给予构造器代表不同算法的函数，就配置了不同的算法。

函数也可以作为类的实例变量。这样在类中，直接维护代表算法的函数也成为可能。从类型声明可以看出，ContextFP中的实例变量strategy就是一个函数。

一等函数的概念使得函数获得了更高的地位，使得函数的灵活性大大增加。在很多场景下直接使用函数会是更直接自然的选择。面向对象编程范式，赋予了对象更高的地位。但是，如果给予函数“正常”一些的地位，可以简化不少问题。设计模式中的不少模式存在都是由于函数的使用限制，需要使用在使用类包裹函数。类似的例子还有模版方法模式（Template method）。

### 柯里化函数（Curried Function）

柯里化函数（Curried Function），是指接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，该函数返回一个接受余下参数的新函数。这个名词来源于逻辑学家：Haskell Curring。编程语言Haskell也取自这位逻辑学家的名字。

抽象工厂模式（Abstract Factory）提供了一种方式，可以将一组具有同一主题的单独的工厂封装起来。在正常使用中，客户端程序需要创建抽象工厂的具体实现，然后使用抽象工厂作为接口来创建这一主题的具体对象。客户端程序不需要知道（或关心）它从这些内部的工厂方法中获得对象的具体类型，因为客户端程序仅使用这些对象的通用接口。抽象工厂模式将一组对象的实现细节与他们的一般使用分离开来。

函数柯里化可以用于制造一系列相关的函数。这和抽象工厂模式的目的类似。仍然如前例，先使用Swift实现经典的抽象工厂模式。首先，我们给出一个产品族，包括两个牌子，两种设备类型的四个组合：

```
	// Abstract Product
	protocol Product {
  		var brand: String { get }
  		var name: String { get }
	}

	protocol Phone: Product { }

	protocol Pad: Product { }

	// Concrete Product
	class ApplePhone: Phone {

  		var brand: String {
    		return "Apple"
  		}
  
  		var name: String

  		init(name: String) {
    		self.name = name
  		}
	}

	class SamsungPhone: Phone {

  		var brand: String {
    		return "Samsung"
  		}

  		var name: String

  		init(name: String) {
    		self.name = name
  		}
	}

	class ApplePad: Pad {

  		var brand: String {
    		return "Apple"
  		}
	  	var name: String

  		init(name: String) {
    		self.name = name
  		}
	}

	class SamsungPad: Pad {

  		var brand: String {
    		return "Samsung"
  		}

  		var name: String

  		init(name: String) {
    		self.name = name
  		}
	}
```

然后，我们需要抽象工厂模式，来为创建这些同一主题的产品提供易于使用，方便扩展的方法：

```	
	// Abstract Factory
	protocol AbstractFactory {
  		func createPhone() -> Phone
  		func createPad() -> Pad
	}

	// Concrete Factory
	class AppleFactory: AbstractFactory {

  		func createPhone() -> Phone {
    		return ApplePhone(name: "iPhone 6S")
  		}

  		func createPad() -> Pad {
    		return ApplePad(name: "iPad Air 2")
  		}
	}

	class SamsungFactory: AbstractFactory {

  		func createPhone() -> Phone {
    		return SamsungPhone(name: "Note 5")
  		}

  		func createPad() -> Pad {
    		return SamsungPad(name: "Note")
  		}
	}
```

再看看如何使用抽象工厂：

```	
	let appleFactory: AbstractFactory = AppleFactory()
	appleFactory.createPad()

	let samsungFactory: AbstractFactory = SamsungFactory()
	samsungFactory.createPhone()
```

对于这个抽象工厂实现来说，虽然加入一个新的设备类型会有很大问题，需要改变抽象工厂的接口，但如果只增加品牌就很好办了。只需在实现一个具体品牌的工厂即可。

然后，我们使用柯里化函数来实现一个工厂方法。这个函数接受创建产品所需要各种参数，返回具体的产品。但我们实际上不会使用这个工厂方法直接创建产品。而是使用柯里化，让这个工厂方法充当一个函数工厂，创建一批可以作为产品工厂的方法。然后，我们可以使用这批产品工厂方法创建具体的产品。

```
	func createProductWithType(type: String)(brand: String)(name: String) -> Product? {
  		switch (type, brand) {
    		case ("Phone", "Apple"):
      			return ApplePhone(name: name)
    		case ("Pad", "Apple"):
      			return ApplePad(name: name)
    		case ("Phone", "Samsung"):
      			return SamsungPhone(name: name)
    		case ("Pad", "Samsung"):
      			return SamsungPad(name: name)
	    	default:
      			return nil
  		}
	}

	let createApplePhone = createProductWithType("Phone")(brand: "Apple")
	let createSamsungPhone = createProductWithType("Phone")(brand: "Samsung")
	let createApplePad = createProductWithType("Pad")(brand: "Apple")
	let createSamsungPad = createProductWithType("Pad")(brand: "Samsung")

	// Use
	let applePhone = createApplePhone(name: "iPhone 6S")
	println(applePhone)
```

与抽象工厂模式相比，使用柯里化函数充当函数工厂，我们可以更轻松地基于一些条件创建一系列工厂方法。

## Swift的语言特性

### 扩展（Extension）

扩展（Extension）扩展是一种向已有的类，枚举或者结构体添加新功能的方法。扩展和Objective-C中的类别（Category）类似，但是与Objective-C中的分类不同的是，Swift中的扩展没有名字。

适配器模式（Adapter）将一个类的接口转接成用户所期待的。适配器使得因接口不兼容而不能在一起工作的类工作在一起。通常适配器有两种实现方法：一种使用继承，创建新类，并继承原有类，同时实现需要兼容的接口；另一种使用组合，创建新类，维护原有类的实例对象，并实现需要兼容的接口。无论哪种方法我们都需要创建一个新的被称为适配器的类。

扩展使得我们多了一种为类增添方法的手段。适配器模式的实质是使得一个已经存在的类，适配另外一个接口。实现接口其实就是实现接口所定义的方法。如果拥有扩展这种工具，就无需适配器模式这种笨重的做法。

下面先试着完成适配器的经典实现。首先，我们写一套配合完美的圆木桩和圆洞。如果圆木桩的直径小于圆洞的直径，毫无疑问，我们可以把圆木桩打入圆洞。

```
	protocol Circularity {
  		var radius: Double { get }
	}

	class RoundPeg: Circularity {

  		let radius: Double

  		init(radius: Double) {
    		self.radius = radius
  		}
	}
	
	class RoundHole {
  		let radius: Double

  		init(radius: Double) {
    		self.radius = radius
  		}

  		func pegFits(peg: Circularity) -> Bool {
    		return peg.radius <= radius
  		}
	}
```

现在我们的工地出现了一根方木桩。我们仍然需要测试这个方木桩是否可以打入圆洞。由于木桩是否能被打入圆洞的测试要求木桩实现了`Circularity`，这时我们就需要一个适配器，将SquarePeg适配为符合Circularity接口的类。适配的结果就是下面的`SquarePegAdaptor`，它实现了Circularity接口。这样，我们可以判断它是否能打入圆洞。

```
	class SquarePeg {
  		let width: Double

  		init(width: Double) {
    		self.width = width
  		}
	}
	
	class SquarePegAdaptor: Circularity {
  	
  		private let peg: SquarePeg

  		var radius: Double {
    		get {
      			return sqrt(pow(peg.width/2, 2) * 2)
    		}
	  	}

  		init(peg: SquarePeg) {
    		self.peg = peg
  		}
	}
```
	
检测结果：

```
	// Test if the square peg is fit with the hole
	let hole = RoundHole(radius: 5.0)
	for i in 5...10 {
  		let squarePeg = SquarePeg(width: Double(i))
  		let peg: Circularity = SquarePegAdaptor(peg: squarePeg)
  		let fit = hole.pegFits(peg)
  		println("width:\(i), fit:\(fit)")
	}
```

然后，我们看看使用扩展是否能简化适配器模式：

```
	// Use extension
	extension SquarePeg: Circularity {

  		var radius: Double {
    		get {
      			return sqrt(pow(width/2, 2) * 2)
    		}
  		}
	}
```

使用扩展，我们无需像适配器模式那样，创建新类充当适配器。而只需要扩展原有类，为原有类添加一个方法，同时就实现了Circularity接口。在使用时，也更简单了，我们只需要直接使用原有类即可：

```
	for i in 5...10 {
  		let peg = SquarePeg(width: Double(i))
  		let fit = hole.pegFits(peg)
  		println("width:\(i), fit:\(fit)")
	}
```
	
由于Swift提供了扩展（Extension）这一新的语言特性，使得处理接口转换这类原来需要适配器模式解决的问题时，更为简单了。

### 运算符重载

解释器模式（Interpreter）定义语言的文法，并建立一个解释器来解释语言中的句子。本质上说，解释器模式是一种语言扩展工具。如果你使用的语言不适合解决某类问题，可以使用解释器模式构建一些新的语法，甚至新的语言用于更为方便地解决特定领域的问题。这是在语言不提供语言级别的对自身扩展时的一种替代。

一些语言会禁止对语言本身进行扩展，例如Java。这可能是因为Java的设计者认为，扩展语言需要专业的技能和良好的语法设计，这通常不是普通程序员可以胜任的。扩展语言这种特性常常会带来一些容易令人混淆的问题，很多描述这类技术的文档，也通常会告诫读者，“如果你不清楚自己在做什么事情，就最好不要使用这个功能”。但另一方面，语言禁止对本身的扩展也就限制了其使用范围，所以，Java语言中才会出现解释器模式这种解决扩展语言自身功能的方案。当然不可避免的，这种解决方案会比有语言级别的支持复杂晦涩。

下面我们会讨论一种扩展语言语法的语言特性：运算符重载。看看有了运算符重载，解释器模式是否能得到简化。运算符重载是多态的一种。运算符（比如+，=或==）被当作多态函数，他们的行为随着其参数类型的不同而不同。这意味着我们可以赋予一个运算符新的含义，使用在新的数据类型上，这就是一种扩展语言语法的特性。

我们仍然分别用解释器模式和运算符重载来解决同一个问题。两个例程的比较可以清晰地说明差别。问题的要求是实现复数的加法和乘法。先用解释器模式实现：

```
	import Foundation

	protocol Expression {
  		func interprete(variables:Dictionary<String, Expression>) ->  ComplexNumber
	}

	struct ComplexNumber: Expression {
  		let real: Double
  		let imaginary: Double

  		init(real: Double, imaginary: Double) {
    		self.real = real
    		self.imaginary = imaginary
  		}

  		func interprete(variables:Dictionary<String, Expression>) -> ComplexNumber {
    		return self
  		}
	}

	struct Plus: Expression {
  		let leftOperand: Expression
  		let rightOperand: Expression

  		init(left: Expression, right: Expression) {
    		self.leftOperand = left
    		self.rightOperand = right
  		}

  		func interprete(variables:Dictionary<String, Expression>) -> ComplexNumber {
    		let left = leftOperand.interprete(variables)
    		let right = rightOperand.interprete(variables)
    		return ComplexNumber(real: left.real + right.real, imaginary: left.imaginary + right.imaginary)
  		}
	}

	struct Multiply: Expression {
  		let leftOperand: Expression
  		let rightOperand: Expression

  		init(left: Expression, right: Expression) {
    		self.leftOperand = left
    		self.rightOperand = right
  		}

  		func interprete(variables:Dictionary<String, Expression>) -> ComplexNumber {
    		let left = leftOperand.interprete(variables)
    		let right = rightOperand.interprete(variables)
    		let resultReal =  left.real * right.real - left.imaginary * right.imaginary
    		let resultImaginary = left.real * right.imaginary + left.imaginary * right.real
    		return ComplexNumber(real: resultReal, imaginary: resultImaginary)
  		}
	}

	struct Variable: Expression {
  		let name: String

  		init(name: String) {
    		self.name = name
  		}

  		func interprete(variables: Dictionary<String, Expression>) -> ComplexNumber {
    		if let variable = variables[name] {
      			return variable.interprete(variables)
    		}
    		return ComplexNumber(real: 0, imaginary: 0)
  		}
	}
```

然后，我们实现解释器。为了使实现更简单，问题更聚焦，我们只实现后缀表达式形式的复数加法和乘法。这是因为对于后缀表达式，我们只需借助栈（Stack）对表达式进行一次遍历，就可以计算出结果，这大大降低了算法复杂度。而对中缀表达式的解释，我们通常也会先将它转化为后缀表达式，再做解释，计算出结果。鉴于这个例子里主要是展示解释器模式，而不是讲解解释器实现，所以，我们仅实现最简单的后缀加法和乘法。

```
	struct Stack<M> {

  		var items = Array<M>()

  		mutating func push(item: M) {
    		items.append(item)
  		}

  		mutating func pop() -> M {
    		return items.removeLast()
  		}
	}

	struct Evaluator: Expression {
  		let syntaxTree: Expression

  		init(expression: String) {
    		var expressionStack = Stack<Expression>()

    		let tokens = expression.componentsSeparatedByString(" ")
    		for token in tokens {
      			if token == "+" {
        			let subExpression = Plus(left: expressionStack.pop(), right: expressionStack.pop())
        			expressionStack.push(subExpression)
      			} else if token == "*" {
        			let subExpression = Multiply(left: expressionStack.pop(), right: expressionStack.pop())
        			expressionStack.push(subExpression)
      			} else {
        			expressionStack.push(Variable(name: token))
      			}
		    }
    		syntaxTree = expressionStack.pop()
  		}

 		 func interprete(variables: Dictionary<String, Expression>) -> ComplexNumber {
    		return syntaxTree.interprete(variables)
  		}
	}
```
	
最后，看看如何使用解释器模式：

```
	let expression = "w x *"
	let sentence = Evaluator(expression: expression)
	var variables = Dictionary<String, Expression>()
	variables["w"] = ComplexNumber(real: 1, imaginary: 2)
	variables["x"] = ComplexNumber(real: 2, imaginary: 4)
	sentence.interprete(variables)
```

由于Swift支持运算符重载，我们可以更简单地使用语言已经提供的特性完成上面例程中相同功能。Swift支持以运算符作为函数名。所以，对于Swift中已经实现的运算符，我们只需要重载两个函数，使运算符可以应用于新的类型：复数类型，即可。

```
	func + (left: ComplexNumber, right: ComplexNumber) -> ComplexNumber {
  		let resultReal =  left.real + right.real
  		let resultImaginary = left.imaginary + right.imaginary
  		return ComplexNumber(real: resultReal, imaginary: resultImaginary)
	}

	func * (left: ComplexNumber, right: ComplexNumber) -> ComplexNumber {
  		let resultReal =  left.real * right.real - left.imaginary * right.imaginary
  		let resultImaginary = left.real * right.imaginary + left.imaginary * right.real
  		return ComplexNumber(real: resultReal, imaginary: resultImaginary)
	}
```

完成运算符重载之后，我们就可以像将两个整数相加一样，使用加法运算符将两个复数相加：

```
	let first = ComplexNumber(real: 1, imaginary: 2)
	let second = ComplexNumber(real: 2, imaginary: 4)

	first + second
	first * second
```

可以看到运算符重载使得扩展运算符的使用范围变得更容易了。这也很容易理解，解释器模式其实就是对语言扩展自身语法这一功能的缺失而设计的设计模式。再次设计很有时候需要绕过很多障碍，很容易比语言的原生支持更为复杂繁琐。

## 总结

这篇短文中，我们回顾了设计模式的概念，并讨论了在现代编程语言背景下，设计模式相关问题的一些变化。然后使用Swift语言演示了5个设计模式和新的语言特性之间的关系。这5个设计模式和语言特性分别是：高阶函数和命令模式，一等函数和策略模式，柯里化函数和抽象工厂模式，扩展和适配器模式，运算符重载和解释器模式。

这5个例子展示了设计模式在现代编程语言中的变化：要么被语言特性更好的代替了；要么变得更为简单了。