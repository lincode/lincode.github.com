---
layout: post
title: FRDModuleManager 模块管理工具
category: project
description: FRDModuleManager 是一个简单的 iOS 模块管理工具。
---
**FRDModuleManager** 是一个简单的 iOS 模块管理工具。如果你发现自己项目中实现了协议 UIApplicationDelegate 的 AppDelegate 变得越来越臃肿，你可能会需要这个小工具；如果你的项目实施了组件化或者模块化，你需要为各个模块在 UIApplicationDelegate 定义的各个方法内留下钩子(hook)，以便模块可以知晓整个应用的生命周期，你可能会需要这个小工具。

FRDModuleManager 可以减小 AppDelegate 的代码量，把很多职责拆分至各个模块中去。这样 AppDelegate 会变得容易维护。

FRDModuleManager 可以使得留在 AppDelegate 的钩子方法被统一管理。实现了协议 UIApplicationDelegate 的 AppDelegate 是我知晓应用生命周期的重要途径。如果，某个模块需要在应用启动时初始化，那么我们就需要在 AppDelegate 的 `application:didFinishLaunchingWithOptions:` 调用一个该模块的初始化方法。模块多了，调用的初始化方法也会增多。最后，AppDelegate 会越来越臃肿。FRDModuleManager 提供了一个统一的接口，让各模块知晓应用的生命周期。在 AppDelegate 中留下钩子，在特定的生命周期调用模块的对应方法。这样将使得 AppDelegate 更简单。对于应用生命周期的使用也更清晰。

项目地址：[FRDModuleManager](https://github.com/lincode/FRDModuleManager) ：
