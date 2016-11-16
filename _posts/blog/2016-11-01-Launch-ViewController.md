---
layout: post
title: iOS 中的 view controller 调用方案
description: iOS 开发中有几种页面调用方案。这篇文章中，我们聊一聊 iOS 中的 view controller 调用方案。简单分析一下它们的实现原理与存在的问题。最后会介绍一下 FRDIntent，一种借鉴了 Android Intent 的页面调用方案。
category: blog
---

iOS 开发中有几种页面调用方案。这篇文章中，我们聊一聊 iOS 中的 view controller 调用方案。简单分析一下它们的实现原理与存在的问题。最后会介绍一下 FRDIntent，一种借鉴了 Android Intent 的页面调用方案。

## iOS 系统方案

首先，iOS 系统本身给我们提供了丰富而完备的 API 用于控制页面转换的各种细节。让我们先看一个直接使用系统 API 调起页面的简单例子。

假设我们在一个名为 FirtViewController 的 view controller 中，需要进入到另一个名为 SecondViewController 的 view controller 中。 我们至少需要做两件事情：

- 首先，需要初始化一个 SecondViewController 的实例 secondViewController。
- 然后，通过 `pushViewController:animated` 方法将这个实例推入到 FirstViewController 所属的 UINavigationController 的页面栈中。

相关代码如下：

```
// in class FirstViewController
SecondViewController *secondViewController = [[SecondViewController alloc] init];
[self.navigationController pushViewController:secondViewController animated:YES];
```

### 问题

初始化一个目标页面，将其放入浏览页面栈，随即显示在前台。这个过程简单而直接，在小型项目中，没什么问题，也是不错的选择。

但是在大型项目中，为了项目结构的健康，我们可能不得不做一些模块化工作。模块化的基本要求是解除依赖。这种直接使用 iOS 系统的方案中，需要调用者自己直接初始化目标页面的实例。这就需要知道目标页面的各种信息，起码需要知道类名和合适的初始化方法。这造成了 view controller 之间的依赖。

如果，在一个大型项目中，需要解除各个模块之间的页面之间的耦合，我们就需要一个间接调用目标页面的方案。

## openURL 方案

在这些间接方案中，有很多是基于 iOS 系统提供的 UIApplicationDelegate 中的一个用于响应 URL 调用的钩子方法。在 iOS 8 中，该方法为：`application:openURL:sourceApplication:annotation:`。在 iOS 9 中被它标记为 `Deprecated`，并提供了一个替代方法：`application:openURL:options:`。我们将这两个方法简称为 `openURL` 钩子方法。

如果，应用注册了一个 URL scheme。任何第三方应用，以及应用自身对该 URL 的调用，都会调起以上的 `openURL` 钩子方法。这就提供了一个在应用间通信的方法。同时这也是目前为止，iOS 系统中，应用间通信的唯一方式。

### URL Routes

为了更便捷地使用 UIApplicationDelegate 的这个钩子方法。iOS 社区中还涌现了很多 URL Routes 项目，例如, [JLRoutes](https://github.com/joeldev/JLRoutes)。这些 URL routes 项目大致都提供了两类功能：

- 对 URL 的模式匹配，使得使用者可以更便捷地归类 URL 和提取 URL 中的参数。
- 提供了闭包注册 URL 的功能。注册之后，任何应用调用 URL，则会调用对应闭包。

基于 UIApplicationDelegate 的 `openURL` 钩子方法，或者更进一步在 `JLRoutes` 这种 URL Routes 库的基础上，我们可以很容易的完成一个 iOS 的页面调用方案。基本过程如下：

- 首先，为不同 view controller 注册不同的 URL，以形成 URL 到 view controller 之间的映射关系。如果使用 JLRoutes，可以在 block 中完成对目标页面的调用。这实际就形成了 URL 到调用代码的映射。
- 调用时，并不用直接初始化 view controller，而是调用一个 URL。
- UIApplicationDelegate `openURL` 钩子方法将被调用。在钩子方法中对注册形成的 URL 到 block 之间的映射关系表进行查询。找到对应的 block，并执行它。block 中的代码应该完成这样的功能：对目标页面的初始化，并将其推入前台。

也有直接为你提供将 view controller 注册到 URL 功能的库。比如，[HHRouter](https://github.com/Huohua/HHRouter)。这种库的目的就更为直接，仅用于页面调用。

### 问题

这些使用了 openURL 钩子方法的 URL Router 的页面调用方案都有两个问题：

- 传递复杂对象困难。这是因为在 opeURL 钩子方法中，调用者和被调用者的通信协议是 URL。URL 本质上是一个一维字符协议。它并不是被设计用来表达具有多个层次的复杂信息。使用 json 或者 xml 这类有结构的格式表达复杂信息会更适用。当然，你可以做一些编码工作，使其可以间接表达更复杂的信息。但是这就存在一些困难，也并不直接。而且 URL 的长度也是有限制的。将 URL 用于标识一个暴露给外部的资源是合适的。iOS 将其用于应用间的通信也是合适的。但在应用内部，信息的沟通更为直接，也更为复杂。URL 这个协议并不适用。

- 没有区分内部调用和外部调用。这是因为 openURL 钩子方法会响应所有应用以注册的 URL 的调用。包括外部应用的调用和来自应用本身的调用。当然，钩子方法的参数 `sourceApplication`，或者参数 `options` 的 key `sourceApplication` 所对应的值，可以帮助你标识出调用方的 bundleID。所以，可以说 iOS 系统提供了区分内部和外部调用的可能性。但是基于此做的 URL routes 库都没有实现这个功能。为了区分内部调用和外部调用，就需要提供两类 URL 的注册：一类用于外部调用；一类用于内部调用。这在 JLRoutes 和 HHRouter 都未见到。这就会导致我们无法区分内部外部调用，只要注册了某个页面调用，即使这个调用只应在内部使用，也会同时开放给外部。

这两个缺陷都是由于对 openURL 钩子方法的理解不当。openURL 钩子方法被设计用于在应用之间进行通信。如果，将其用于应用内部的通信，我们就需要问一下这个 API 是否还适用呢？起码，在我们的项目实践中发现，仅仅用 URL 协议在应用内部通信的话，会感觉受到限制。

## FRDIntent

由于 URL Routes 存在一些使用上的不便和问题，我们试图寻找另外的解决方案。Android 系统给出了一个不错的方案。Android 的 Intent 是一个消息对象，可用于调起代表页面的 Activity 和代表服务的 Service。并且 Android 的 Intent 还是一个系统的级的消息对象。可以用于调起第三方应用，或者系统级的页面或者服务。

### 应用内页面调用：FRDIntent/Intent

[FRDIntent](https://github.com/douban/FRDIntent) 是一个用于解决应用内应用外页面调用的库。FRDIntent 也是该库中一个重要对象的名字，它是一个消息对象，用于启动 UIViewController。这是对 Android 系统中的 Intent 的模仿。当然，相对于 Android Intent，FRDIntent/Intent 做了极度简化。这是因为 FRDIntent/Intent 的使用场景更为简单：只处理应用内的 view controller 间跳转。

在 `FRDIntent` 库中，使用一个名为 FRDIntent 的对象传递消息。这是整个库的核心类，所以库的名字也是用这个类命名的。和 URL 相比，应用内部使用对象表达信息更为直接和便利。由此，带了 `FRDIntent` 的诸多优势：

- 充分解耦。调用者和被调用者完全隔离，调用者只需要依赖协议：FRDIntentReceivable。一个 UIViewControlller 符合该协议即可被启动。
- 对于“启动一个页面，并从该页面获取结果”这种较普遍的需求提供了一个通用的解决方案。具体查看方法：startControllerForResult。这是对 Android 中 startActivityForResult 的模仿和简化。
- 支持自定义转场动画。
- 支持传递复杂数据对象。

### 应用外通信：FRDIntent/URLRoutes

FRDIntent/URLRoutes 是一个 URL Router。它提供了将 block 注册到 URL，建立关联的能力。然后，通过在 UIApplicationDelegate 的 openURL 钩子方法中放置代码，获知外部的 URL 调用，就可以调用对应的 block。可以看出，FRDIntent/URLRoutes 和社区已经存在的诸如 JLRoutes 和 HHRouter 这样的库的功能和目的差别不大。

### FRDIntent/Intent 和 FRDIntent/URLRoutes

FRDIntent/Intent 提供了一个消息对象作为通信中介。URL 只是该消息对象一个标识。除此之外，还可以为消息对象配置目标页面的初始化参数，转场动画。这样，就充分满足了页面切换过程中所需要的携带的信息。FRDIntent/Intent 只用于管理应用内部的页面调用。理论上，如果 iOS 系统开放更多 API，应该也可以像 Android Intent 那样，实现一个应用间传递的消息对象。但由于 iOS 系统只提供了 openURL 这一种方式用于应用之间通信，所以现在 FRDIntent/Intent 只能用于应用内部。

FRDIntent/URLRoutes 还是保持了和社区中其他 URL Routes 库基本一致的思想，仍然使用 URL 作为通信协议。但 FRDIntent/URLRoutes 只用于管理应用向外部暴露的页面调用。通过 FRDIntent/URLRoutes 向外暴露一个页面调用服务，其实就是为一个 block 注册一个 URL。该 block 内执行的代码仅仅是发出一个 FRDIntent 消息对象，以调起一个页面。即外部调用在应用内部通过 FRDIntent 落地，最终启动 view controller。也就是说外部调用实际上是通过内部调用实现的。

可以看出，在 FRDIntent 中，FRDIntent/URLRoutes 和 FRDIntent/Intent 一起相互配合完成了对应用外和应用内的页面调用。

## 总结

FRDIntent/Intent 借鉴了 Android 系统，为 iOS 提供了一个类似 Android Intent 的消息对象。用于处理应用内的页面调用。同时，封装了一个 FRDIntent/URLRoutes 用于处理外部调用。

FRDIntent 较基于 openURL 钩子方法的一些 URL Routers 不同之处在于，区分了内部和外部调用的处理方式。外部调用仍然基于 openURL 钩子方法。内部调用使用消息对象。这个区分改善了 openURL 方案的一些问题。为在大型项目中，解除 view controller 之间的耦合提供了一个选择。

## 相关项目仓库

- [FRDIntent](https://github.com/douban/FRDIntent)
- [JLRoutes](https://github.com/joeldev/JLRoutes)
- [HHRouter](https://github.com/Huohua/HHRouter)