---
layout: post
title: 豆瓣的混合开发框架 -- Rexxar
description: 最近两年，我们在移动混合开发实践方面做了一些努力。主要产出是一个名为 Rexxar 的项目。现在，我们把这个项目开源了。一方面，是为了给大家提供一些借鉴；另一方面，是为了提高这个项目的质量。接受大家的意见和建议。
category: blog
---

## 背景

豆瓣在2014年推出一个主应用：[豆瓣 App](https://www.douban.com/doubanapp/)。这个主应用慢慢成长，逐渐覆盖了豆瓣在 Web 上的大部分功能。随着项目的扩大，产品线的扩展，和产品需求增长，豆瓣App 同时成为了一个需要提供 iOS，Android 和移动 Web 页面的多平台支持的服务。工程技术团队为了更从容地应对这种状况，开始投入较大的精力提高团队的开发效率。混合开发是其中主要的措施之一。

由于项目已经发展到一定程度，我们并不希望推倒以往的开发方式，也没有一切从头来的野心和勇气。只是希望在不影响 App 的性能前提下，在合适的地方使用 Web 技术部分地提高开发效率。而豆瓣 App 中又确实存在部分页面是重度展示，却轻度的交互的页面。这些页面恰恰比较适合使用 Web 技术来实现。

经过团队的一些努力，App 中部分页面已经使用 Web 技术实现，并在取得了不错的效果。工程师使用 Web 技术开发的页面可以部署到两个平台，开发效率得到了实质性提高。就算不提热更新，减少 Android 项目方法数这种附带的好处，我们都已喜欢上这项技术，决定推动在豆瓣移动开发中的推动混合开发的使用。

现在，我们将这个过程的主要产出：**Rexxar** 这个项目开源。一方面，是为了给大家提供一些借鉴的方向；另一方面，是为了提高项目本身的质量。我们知道还存在不少问题。所以，会悉心接受大家的意见和建议。

## 混合开发

混合开发的直白的解释是 Native 和 Web 技术都要用。但形式上，应用仍然和浏览器无关，用户还是需要在 App Store 和 Android Market 下载 App。只是在开发时，开发者以 Native 代码为主体框架，在合适的地方部分使用 Web 技术。比如在 UIViewController 中放置一个 UIWebview（一个浏览器引擎，只拥有渲染 HTML，CSS 和执行 JavaScript 的核心功能）。这样，部分用户界面就可以使用 Web 技术实现。

促使我们在移动开发中使用 Web 技术主要动力在于，相比于 Native 技术，Web 技术具有诸多优势：

- HTML，CSS，JavaScript 的组合被证明在用户界面开发方面具有很高的效率。
- 统一的浏览器内核标准，使得 Web 技术具有跨平台特性。iOS 和 Android 可以使用一套代码。
- 可越过发布渠道自主更新应用。

这些优势都和开发效率有关。Web 技术具有这些优势的原因是，Web 技术是一个开放标准。基于开放的标准发展出来的庞大生态，而且这个生态从 PC 时代发展至今已积累多年，开发者可以利用生态中产出的各种成果，而省去很多重复工作。

在大型移动应用的开发中，项目代码庞杂，通常还是 iOS，Android， 移动 Web 和 桌面 Web 全平台支持。这种情况下，更高的开发效率就成为了开发者不得不考虑的问题。这也是为何虽然移动平台的 Web 技术在使用范围和性能上有诸多劣势，仍然有很多开发者付出努力探索如何在移动开发中使用 Web 技术。

我们也是基于以上各种考虑，在项目中实践了一些混合开发技术。

## Rexxar

**Rexxar** 是一个针对移动端的混合开发框架。现在支持 Android 和 iOS 平台。并有一个 Web 基础库。

团队中喜欢玩魔兽的同学将该项目命名为 Rexxar（《魔兽世界》中人物，出生于卡利姆多大陆的菲拉斯，同时具有雷骨兽人和南部菲拉斯野生食人魔血统）。

各平台代码仓库地址：

- Rexxar Web：[https://github.com/douban/rexxar-web](https://github.com/douban/rexxar-web)
- Rexxar iOS：[https://github.com/douban/rexxar-ios](https://github.com/douban/rexxar-ios)
- Rexxar Android：[https://github.com/douban/rexxar-android](https://github.com/douban/rexxar-android)

Rexxar 主要由以下三部分组成：

- Rexxar Route，我们使用 URL 来标识每一个页面。在 App 中通过指明 URL 跳转到此页面。所以，需要一个路由表，可以根据 URL 找到一个 Rexxar Web 的对应资源来正确展示相应页面；

- Rexxar Web，前端代码库，由 HTML、CSS、JavaScript、Image 等组成，用来提供在移动客户端使用的用户页面；

- Rexxar Container，一个前端代码的运行容器。它其实是一个内嵌的浏览器(WebView)，我们为内嵌浏览器提供了一些必要的原生端支持，包括 API 的 OAuth 授权、图片缓存等；现在有 Android 和 iOS 两个版本的实现。

在项目实践中，Rexxar Web 和 Rexxar Route 由一个项目实现，并部署于同一个 Web 项目中。

### Rexxar Route

Rexxar Route 比较简单，只需要表达一个路由表即可。我们使用了一个 json 文件来表达路由表。给出一个路由表的例子：

```
    {
        count: 4,
        items: [{
            remote_file: "https://img1.doubanio.com/dae/rexxar/files/orders/orders-70dbdbcb1c.html",
            uri: "douban://douban.com/orders[/]?.*"
        }, {
            remote_file: "https://img1.doubanio.com/dae/rexxar/files/related_doulists/related_doulists-1d7d99e1fb.html",
            uri: "douban://douban.com/(tag|tv|movie|book|music)/(\w+)/related_doulists[/]?.*"
        }   ],
        deploy_time: "Fri, 04 Mar 2016 11:12:29 GMT
    }
```

我们发布的每个版本的 App 安装包都会包含最新版本的 routes.json 文件。在 App 启动时，都会尝试下载最新版本的 routes.json。在遇到无法解析的 URL 时，也会去下载新版 routes.json。

### Rexxar Web

Rexxar Web 是 Rexxar 前端实现。Rexxar Container 的实现和 Rexxar Web 的实现是分离的。Rexxar Container 对 Rexxar Web 使用何种技术实现并不关心。所以，你可以选择自己的前端技术和 Rexxar Container 进行组合。比如，我们在业务层选择了 React 作为前端开发框架。

Rexxar Web 包括了三部分内容：

#### 工具

一套开发 Rexxar Web 所需的打包，调试，发布工具。

#### 公共的前端组件

- 通用的错误处理、Loading等效果；
- 页面点击反馈效果；
- List 的支持；
...

#### 对 Rexxar Container 实现的 Widget 的调用

- ActionBar 的 title 定制
- ActionBar 的 button 定制
- Dialog
- 下拉刷新
- Toast

有了这些组件，我们日常产品开发的难度就降低了。普通移动开发工程师经过一段时间的学习，也可以像前端工程师一样，以 Rexxar 为工具为 App 做一些产品开发了。这部分可以视为一个纯粹的前端项目。

### Rexxar Container

我们使用混合开发技术提高开发效率的一个前提是，尽量不损伤 App 的使用体验。基于这个前提，在 Native 和 Web 如何分工方面我们做了一些尝试。首先，为了保证使用体验，我们把 App 里页面切换留给了 Native。这样，每个页面（Controller 或者 Activity）都是一个 Container。Container 内嵌一个浏览器内核。页面内的功能和逻辑在 Native 和 Web 之间如何分工呢？我们尝试过有几种策略：

- 纯浏览器方案：也就是 Native 除了扔给内嵌浏览器一个 url 地址之外，就没有不做任何事情了，剩余的事情都由 Web 技术完成。这和用 Safari 或 Chrome 等普通浏览器打开一个网页并没有太多区别。只是我们固定了访问的地址。

- 前端模板渲染容器方案：这种方案大部分事情由 Native 完成，Web 部分只是负责页面元素的呈现，不参与页面界面之外的其他部分。我们在客户端存储了一个 HTML 作为 UI 模板。Native 代码负责获取数据，向 HTML 文件模板中填入动态数据，得到一个可以在内嵌浏览器渲染的 HTML 文件。这个过程有点类似于 Web 框架里模板渲染库（例如，Jinja2）的作用。

- Rexxar Container 方案：Rexxar 采用的方案介于上述两种方案之间。Rexxar Container 同样提供了一个运行前端代码的容器。它也是一个内嵌的浏览器(WebView)。只是，我们并不是扔给内嵌浏览器一个 url 地址就放手不管了，而是对内嵌浏览器包装了很多功能。

Rexxar Container 方案中，Container 需要实现以下功能：

- Rexxar Route 路由表的更新，已经在客户端的保存；
- 为 Rexxar Web 前端代码发出的 API 请求提供包装。带上必要的 OAuth 参数；
- 缓存 Rexxar Web 前端代码所需要的静态文件，包括 HTML、CSS、JavaScript、Image(图片素材)等；
- 缓存 Rexxar Web 中所需要加载的资源文件，例如图片等；
- 通过协议为 Rexxar Web 提供一些原生支持的功能。

这种实现方案，是基于保证使用体验的前提下，尽量让 Web 技术多做一些事情的考虑。

#### Rexxar Container 和 Rexxar Web 之间的交互

混合开发实践中，一般都会涉及到 Native 和 Web 如何通信的问题。这是因为我们把一件事情交给两种技术完成，那么它们之间便会存在有一些通信和协调。通常会使用 JSBridge(Android: [JsBridge](https://github.com/lzyzsd/JsBridge)，iOS：[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)) 来实现 Native 和 Web 的相互调用。

但在 Rexxar 中，我没有使用类似 JsBridge 这样的方案。而是从 Rexxar Web 发出 HTTP 请求的方式，由 Rexxar Container 截获的方式进行通信。Native 和 Web 之间协议是由 URL 定义的。Rexxar Web 访问某个特定的 URL, Rexxar Container 截获这些 URL 请求，调用 Native 代码完成相应的功能。

例如，Rexxar 中 UI 相关的功能的协议如下：

- 请求 douban://rexxar.douban.com/widget/nav_title，可以定义 Navigation Bar Title。
- 请求 douban://rexxar.douban.com/widget/nav_menu，可以定义 Navigation Bar Button。
- 请求 douban://rexxar.douban.com/widget/toast，可以出现一个消息通知 toast。

Rexxar Web 具体前端实现是在 DOM 中加入一个 iframe 来加载此 URL，以来完成对 Rexxar Container 的通知。

将 Native 和 Web 的通信以协议的形式规范起来，是因为我们希望 Native 和 Web 之间的通信是可定义的，可控的。有这种期望的原因是，我们以 Rexxar 完成的页面，不仅仅使用在 App 内，还会使用在移动 Web 页面上。我们的移动站点，特别是分享到外部（如微信，微博）的页面也希望复用 Rexxar 在 App 内的工作成果。如果，任由开发者自由的定义过多的依赖于原生实现的功能，那么我们就无法顺利地迁移到移动 Web 上去。标准浏览器并不支持 JSBridge 的大部分功能。可以看到我们已经实现的协议，大部分在移动 Web 是被可以忽略的（比如，nav_title, nav_menu），或者我们也可以较容易地以移动 Web 支持的形式再实现一次（比如，toast）。Rexxar 中的前端业务代码并无需太多改动即可迁移到 Web 端。

#### Rexxar Container 的技术实现

Rexxar Container 主要的工作是截获 Rexxar Web 的数据请求和原生功能请求。Rexxar Container 截获请求之后，做相应的反应。这种 Native 和 Web 的交互被抽象成三种接口:

- Decorator：修改数据请求。例如，数据请求加上 OAuth 认证信息。
- Widget: 调用某些原生 UI 组件。例如，调起一个 Toast。
- ContainerAPI：要么给出一个 Native 的计算结果。例如，给出当前位置信息。

这三种接口都是由 Rexxar Web 发起某种形式的 URL 调用的。这样，Rexxar Web 的业务代码在 App 的 Rexxar Container 内工作方式，就和在普通浏览器里差别不大。我们只是在 Web 技术的基础上做了一些拓展，保留了大部分 Web 原有的编写和运行方式。代码都是标准 Web 式的，没有为原生移动开发做太多定制。可以顺利移植到 Web 平台，在各种浏览器中都可以正确运行。以 URL 作为协议，也为 Web 和 Native 划定了清晰的边界，和数据传递方式。

我们为 iOS 和 Android 各开发了一个 Rexxar Container。iOS 和 Android 平台截获请求的方式由于平台差异并不完全相同。但本质上都是在 Web 和 Native 之间实现了一个 Proxy。Web 发出的请求会被 Proxy 预先处理。要么是修改后再发出去，要么是由 Rexxar Container 自己处理。

具体的实现可以参看两个平台的项目代码。

### Rexxar 页面执行过程

![Rexxar 工作流](/images/blog/Rexxar.png)

例如，客户端接到一个页面请求，要打开一个 URL：douban://douban.com/movie/1292052。Rexxar 的工作流如下：

1. 根据 URL 查询本机缓存的路由表，看是否能够找到对应的资源记录(一般是一个 HTML 文件)。如果找到不到，请求 Rexxar Route 服务，获得最新的全量路由表，更新本地缓存，找到对应的资源记录；

2. 根据路由表指示的 HTML 文件的路径，看本地是否找到对应的文件。如果找不到，请求 Rexxar Web 资源服务器，更新本地缓存；

3. 在 Rexxar Container 里展示该 HTML 文件；如有需要，在 Container 中请求 Image，先检查本地缓存。如不存在，请求 Rexxar Web 资源服务器；

4. Rexxar Web 前端代码在 Container 里继续执行，发出 API 请求，有 Rexxar Container 代理这些请求，为 API 请求添加 OAuth 验证；

5. Rexxar Web 前端代码继续执行，根据 API 返回的结果，展示响应的页面，可能会请求 CDN 的图片等；

6. Rexxar Web 前端代码继续执行，如果需要修改 NavigationBar 等原生界面，可能通过定义好的协议向 douban://rexxar.douban.com 发送数据。Rexxar Container 拦截请求，按定义好的协议作出反应，例如，修改 NavigationBar 上的按钮。

## Rexxar 的问题

#### 性能

混合开发的问题在于，Web 的性能没法和 Native 相比。这种状况可能会长期存在。因为，前端代码运行于内嵌浏览器之上，和直接调用原生系统相比，理论上总会存在性能上的差距。我们现在基本是以规避的方式面对性能问题：即性能问题会明显影响到用户体验时，我们就不使用 Rexxar 来做，而是使用传统 Native 的方式老老实实写两份 Native 代码，一份 iOS，一份 Android。当然，这就限缩了 Rexxar 的使用范围。

在 Rexxar iOS 中，我们做了使用 WKWebView 替代 UIWebView 的尝试。但是现在看起来这会是一个长远目标。WKWebView 在速度和内存消耗上都优于 UIWebView。但 WKWebView 并不完善。对于 Rexxar iOS 而言，最重要的缺陷是不支持使用 NSURLProtocol 截获 WKWebView 中发出的网络请求。所以在现有的 Rexxar 的实现中，并没有使用 WKWebView。但是，我们会持续努力，以寻找切换至 WKWebView 的可能性。

#### 错误报告

我们在应用中使用 Rexxar 之后，在收集到的 Crash Report 中，JavaScript 的相关错误，和浏览器相关的错误开始增加。而对这类错误，由于移动应用的使用环境更为复杂，错误报告经过了 JavaScript 引擎，原生系统两层之后，给出的错误信息并不够明确。我们在这方面的经验也并不多，导致我们还没有很好的办法降低这类错误。这对提高 App 的稳定性带来了问题。

## 总结

Rexxar 这个混合开发框架在豆瓣移动开发中使用，确实在一定程度上提高了我们的开发效率。以前一个页面需要 iOS 和 Android 两位工程师各开发一遍，现在只需要一位工程师写一次前端代码，甚至还可以应用到移动 Web 上去。虽然 Rexxar 仍然存在一些问题，和使用上的限制。但是在有限的使用中，我们仍然收获不少。所以，在未来我们应该会持续推动 Rexxar 在豆瓣移动开发中的使用。

希望这个 Rexxar 这个开源项目对大家能起到一点启示效果。并得到大家的反馈和建议，帮助我们提高。
