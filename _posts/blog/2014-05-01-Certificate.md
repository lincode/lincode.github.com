---
layout:    post
title:     iOS 开发中的证书和钥匙
category:  blog
description: iOS 开发中的证书和钥匙
tags: iOS
---

# iOS 开发中的证书和钥匙 
## 公钥和私钥

公钥和私钥，基于 RSA 算法，是不对称加密方式，相对于对称加密（如，用户名和密码）而言是一种改进。公钥，顾名思义，就是给大家用的，任何人都可以获得。公钥用于加密内容或者验证数字签名。私钥，只能由钥匙持有者所有。私钥用于解密内容或者签写数字签名。公钥与私钥的作用是：用公钥加密的内容只能用私钥解密，用私钥加密的内容只能用公钥解密。

例如，我要给你发一封加密邮件。我必须有你的公钥，我用你的公钥给邮件加密，这样就只能保证邮件不被别人看到，而且不被篡改。因为只有你可以用你自己的私钥解密。

另外，我可以用我的私钥给邮件加密，这就是签写数字签名。发给你之后，你可以用我的公钥解密，这就是验证数字签名。这样就能保证邮件是我发的。

以公钥为基础，可以组成数字证书，用于验证证书持有者或者发送者身份。可以做到如下两点：

* 保证数据是有签名者发送的，签名者不能否认。
* 保证数据自签发后到收到为止，没有被篡改过。


## Apple 的证书，钥匙，可信任服务

### 证书，钥匙，可信任服务的概念

证书，钥匙，可信任服务是一个函数和数据结构集合，用于认证和授权用户和进程使用钥匙和证书。因为在 OS X 和 iOS 中，证书（certificate）和钥匙（key）被存储在钥匙扣（keychain）里，这个 API 里的很多功能必须和 Keychain Services API 共同使用。

这一章讨论一些你需要理解的概念，以便理解如何使用证书，钥匙和可信任服务 API。

### 证书，钥匙，和身份认证

一个数字证书是一个被用于验证证书持有者或者发送者身份的数据集合。例如，一个证书包含这样一些信息：

* 证书发布者
* 证书持有者
* 有效期（证书在这个时期之前或之后无效）
* 证书拥有者的公钥
* 证书扩展，包含一些额外信息，例如，允许使用与这个证书关联的私钥
* 从证书权威机构那获得的数字签名，确保证书没有被修改并指明身份认证的所有者

每一个证书都是通过另外一个证书认证的，这创建了一个终结于根证书的证书链。证书的发布者被成为证书权威机构（certification authority CA）。根证书的拥有者是根证书权威机构。

每一个公钥是一个公私钥对的一半。如名字所暗示的那样，公钥可以被任何人获得，但是私钥只能为钥匙所有者秘密所有。私钥加密的数据可以通过公钥解密，反向也可以。为何加密和解密，因此，一个用户必须同时拥有公钥（通常在包含在证书中）和私钥。一个证书和和它相关联的私钥合在一起被称为身份认证（identity）。证书，钥匙，和可信任的服务包括寻找和身份认证关联的证书或者钥匙，和给定一些搜索条件寻找身份认证的功能。搜索条件包括是否允许使用钥匙。

在 OS X 和 iOS 中，钥匙和证书被存储于钥匙扣中，钥匙扣是一个提供了安全存储（如，加密）的用于存储私钥和其他没有加密的涉密数据的数据库。证书，钥匙，和可信任服务的搜索证书，钥匙，和身份认证的功能都用到了钥匙扣。在 OS X 系统，你可以使用钥匙扣访问工具，看钥匙扣的内容和检验证书的内容。

### 证书，钥匙，和可信任服务，和 CDSA

在 iOS 中，钥匙扣服务 API 提供所有操作钥匙扣条目的可能的操作。

在 OS X 中，证书，钥匙，和可信任的服务和其他的安全 API 建立在开源共同数据安全构架（CDSA）之上，它的编程接口是共同安全服务管理（CSSM）。

证书，钥匙，和可信任服务 API 提供的功能执行应用所需的大部分操作，包括生成钥匙对，获取和身份认证相关的证书或者私钥，从系统获取根证书，确认证书合法性，评价信任。相关 CSSM API 提供更多能力，有利于一些特别的应用，例如设计用于管理计算机或者网络安全的应用。基于这个原因，证书，钥匙，和可信任的服务 API 包括一些功能，返回或创建 CSSM 结构，以至于你可以自由移动于证书，钥匙，和可信任的服务和 CSSM 之间。

## iOS 开发中的实例

### 钥匙串访问工具（Keychain Access）
钥匙串访问工具（Keychain Access），用于创建证书请求、安装证书、导出私钥等。

通过钥匙串访问工具生成， CertificateSigningRequest.certSigningRequest 文件，以备向 Apple 请求证书。在生成 CertificateSigningRequest.certSigningRequest 文件的同时，钥匙串也生成了一对公钥和私钥。所以 CertificateSigningRequest.certSigningRequest 文件应该包含公钥。

### Certificate file 
登录开发者中心：https://developer.apple.com/ios/manage/overview/index.action

申请证书的先决条件是拥有 Apple Developer Program membership。申请用于发布应用到 AppStore 的 distribution 版的证书需要更高的权限： Team Agent 或者 admin。

上传 CertificateSigningRequest.certSigningRequest，Apple 会颁发一个证书给你，证书是一个 *.cer 结尾的文件。

对照上面数字证书的概念，这个证书：

* 证书发布者：Apple Worldwide Developer Relations Certification Authority
* 证书持有者：申请证书的开发者
* 有效期：
* 证书拥有者的公钥
* 从证书权威机构那获得的数字签名

证书的作用是向 Apple 表明，你的数据包，例如，包含你的代码的应用包是由你签名的。需要这个证书的原因是 Apple 的某些服务（例如，真机调试，使用 Push Notification Server，AppStore 发布应用）需要你是合法的开发者，才有权限使用这些服务。

除了 provision file 中用到的证书，这个证书用于真机调试，和 AppStore 发布应用。而使用到其他服务还需要申请 push notificaiton certificate，passbook certificate 等。

证书的原理是，开发者使用自己的私钥为数据签名。

例如，发布应用，就需要将应用的二进制包签名，这样 AppStore 使用证书可以确认这个包是由特定的有权限的开发者
所发布的，确认完成后才会允许应用上传。

例如，发送推送提醒，需要向 Apple Push Notification Server 表明自己有权限向某个应用推送提醒。开发者就需要用自己私钥为推送的数据签名，Apple 则用证书验证签名。

### Provision file
在开发者中心，创建一个 provision file。需要先创建 App ID，标识一个应用。如果是用于开发的 provision file 还需要创建一批设备，用于指明哪些设备可以用于测试开发版。

所以，Provision file 包含了以下三个内容：

* 数字证书
* 对应的 App ID
* 设备列表（可选）


## 一些实际问题

### 为何需要 Code Signing？

Code Signing 你的应用使得操作系统可以识别出是谁为你的应用签名了，并确保应用的内容没有被修改。但需要注意，**资源文件并没有被签名**，也就是说如果你替换了图片，nib 等资源文件，这个变化并不会破坏签名的有效性。

### 换一台电脑 Code Signing？

有人使用导出私钥的方法。但这其实是有风险的。因为私钥应该由特定权限的人妥善保存。

团队开发中，如果需要不同人都可以发布应用。就需要共享私钥。可以从钥匙扣访问工具（Keychain Access）中导出相应证书的私钥，通常是 .p12 格式文件。将 .p12 和证书发送给团队中的成员，他们安装了私钥和证书，就可以发布应用了，但是他还是要使用私钥所有人的账号登录来发布，实际上看起来也就是多了一台可以发布应用的电脑。

再强调一次，这是有风险的。

可以使用导出 development profile 的方法：
Xcode → Preferences → Accounts，选择你要导出的 Apple ID，点击左下角的齿轮，选择 “Export Accounts”。你可以导出一个 .developerprofile 文件，这个文件是需要加密。文件内包括:

* Development certificates
* Distribution certificates
* Provisioning profiles

这比直接传私钥要安全得多。
导入 developprofile 文件的方法同上，只是选择 “Import Accounts”。

### 如果 Certificate 过期了？

* Push Notification Certificate
你将不能再向你的应用发送推送消息。

* Pass Type ID Certificate
如果证书过期，已经安装到用户机器上的 pass 将继续正常工作。可以，你将不能再签发新的 pass 或者更新现有的 pass。如果你的证书被废除了（revoke），你的 pass 将不能正常工作。

* iOS Distribution Certificate （App Store）
如果你的 iOS Developer Program membership 仍然有效，你现有的应用不会受到影响。但你将不能再提交新的应用或者更新已有的应用。即使 iOS Developer Program membership 已经失效，已经提交的应用仍然不会受到影响。

* iOS Distribution Certificate (In-house, Internal Use Apps)
如果 inhouse 证书过期，用户将不能再运行这个证书签发的应用。你必须发布一个用新的证书签发的新版本。

### 如果 Revoke Certificate？

当你不再需要证书或者当你因为其他 code signing 原因需要重新创建证书时，你可以废除证书。如果你怀疑你的证书被破解了，你也可以废除证书。如果你时一个公司的 team admin，你可以废除以后不在项目内工作的组内成员的开发证书。废除证书可能会使的 provisioning file 失效，所以废除之前要慎重阅读相关文档。

任何角色（包括 team agent）也无法废除 Passbook 证书，这需要向 product-security@apple.com 发送废除证书的请求。其余的，可以在 Member Center 中操作，当根据不同角色有不同权限。

* Revoke Push Notification Certificate
你将不能再向你的应用发送推送消息。

* Revoke Pass Type ID Certificate
这需要向 product-security@apple.com 发送废除证书的请求。如果你的证书被废除了（revoke），你将不能签发新的 pass，已经发送给用户的也不能正常工作。

* iOS Distribution Certificate （App Store）
现有应用不会受到影响，但你将不能提交新的应用。

* iOS Distribution Certificate (In-house, Internal Use Apps)
如果 inhouse 证书过期，用户将不能再运行这个证书签发的应用。你必须发布一个用新的证书签发的新版本。


## 参考文档

* [Certificate, Key, and Trust Services Concepts](https://developer.apple.com/library/mac/documentation/security/conceptual/certkeytrustprogguide/02concepts/concepts.html#//apple_ref/doc/uid/TP40001358-CH204-TP9 "certificate")

* [Maintaining Your Signing Identities and Certificates](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingCertificates/MaintainingCertificates.html#//apple_ref/doc/uid/TP40012582-CH31-SW32 "")


