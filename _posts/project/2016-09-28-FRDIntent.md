---
layout: post
title: FRDIntent 页面跳转管理工具
category: project
description: FRDIntent 是一个 iOS 系统的页面跳转工具。
---
**FRDIntent** 包括两部分 FRDIntent/Intent 和 FRDIntent/URLRoutes。它们分别可以用于处理 iOS 系统中的应用内和应用外的 view controller 调用。

- FRDIntent/Intent 是一个消息传递对象，用于启动 UIViewController。可以认为它是对 Android 系统中的 [Intent](https://developer.android.com/guide/components/intents-filters.html) 的模仿。当然，FRDIntent/Intent 对 Android Intent 而言，做了极度简化。这是因为 FRDIntent/Intent 的使用场景更为简单：只处理应用内的  view controller 间跳转。

- FRDIntent/URLRoutes 是一个 URL Router。通过 FRDIntent/URLRoutes 可以用 URL 调起一个注册过的 block。

项目地址：[FRDIntent](https://github.com/douban/FRDIntent)。
