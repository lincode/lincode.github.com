---
layout: post
title: 介绍几个值得一玩的AR/VR项目
description: 介绍几个值得一玩的AR/VR项目。其中的几个头显对消费者没有吸引力。但对于学习和理解AR/VR的基本原理，或者开发一些早期项目是有帮助的。它们为探索AR/VR开了一道门。现在这个时代对创新更友好了。技术更为开放，基础设施更完善，信息更易获取。十年前我不会想自己组装一台智能手机。但是现在你如果想组装一台AR或者VR头显，并实现一些有趣的早期项目，其实难度并不大。当然这距离将它们提供给普通消费者使用还有很长距离。
category: blog
---

介绍几个值得一玩的AR/VR项目。其中的几个头显对消费者都没有吸引力。但对于学习和理解AR/VR的基本原理，或者开发一些早期项目是有帮助的。它们为探索AR/VR开了一道门。现在这个时代对创新更友好了。技术更为开放，基础设施更完善，信息更易获取。十年前我不会想自己组装一台智能手机。但是现在你如果想组装一台AR或者VR头显，并实现一些有趣的早期项目，其实难度并不大。当然这距离将它们提供给普通消费者使用还有很长距离。

![Headset](/images/blog/ARVR/Headset.jpg)

## Google VR 
Google VR是Google的手机VR项目，提供了一个配合手机实现VR功能的纸制盒子：Cardboard，和制作更精良的版本：Daydream。由于现有手机功能其实已经很不错，而专业一些的VR头显也还处于发展初期。它们的基本原理和Goolge VR这种简易实现仍然差不多，在休闲游戏上的VR体验区别也没有本质区别。 所以用Google VR了解VR是不错的选择。看起来Google在VR上面只是玩票，现在这个项目已经黄了。

![Daydream](/images/blog/ARVR/Daydream.jpg)

### 特点 
* 硬件，是一个手机VR盒子Cardboard，或者贵一点的Daydream头显，都需要依托于手机。现在仍然很容易在淘宝上购得。 
* 软件部分，Google VR由于是Google的项目，支持非常完善。各个开发平台都有。 

* [https://developers.google.com/vr](https://developers.google.com/vr) 
* [https://github.com/googlevr](https://github.com/googlevr) 

## HoloKit 
HoloKit是一个个人项目，和Google的Cardboard类似，用一个带透镜的纸盒和一台支持ARCore的Android手机或支持ARKit的iPhone手机来实现AR效果。这个纸盒设计挺巧妙的，用手机的显示屏和一组反射和透镜实现了AR眼镜的光学显示部分。然后充分利用了手机摄像头，传感器和软件实现了平面识别和定位。 现在这个项目感觉快黄了。无论AR还是VR，借助于手机都不受市场欢迎。但是对于开发和学习，HoloKit仍然是一个很好的低成本的入门头显。因为它起码是一个头显了，虽然简陋了一点。隔着手机屏幕的AR是当下受设备限制而出现的一种非常怪异的交互，会很快消失。

网易出过一个用塑料制作的升级版。但是猜测这个项目黄了。现在已标记为售罄。 

![HoloKit](/images/blog/ARVR/Holokit.jpg)

### 特点 
* 硬件上很简单，手机AR盒子，依托于手机。 在淘宝上可以买到。
* 软件方面，github上的软件项目在定位方面做了一些工作，主要是为了解决Android手机的定位问题。但随着ARkit和ARCore的逐渐成熟。Holokit的软件其实意义不大了，Unity已经完成了对ARkit和ARCore的良好封装。 直接使用Unity和，HoloKit盒子，一台手机就可以开发了。

* [https://holokit.io/](https://holokit.io/) 
* [https://github.com/holokit/holokitsdk](https://github.com/holokit/holokitsdk) 


## Project North Star
Leap Motion主导的一个开源AR眼镜项目。这个项目公开了North Star AR头显的机械部分的STL文件，显示驱动电路板的PCB文件，以及Leap Motion的手势跟踪的代码。光学部分使用和HoloKit一样使用反射。比较酷的是由Leap Motion传感器实现手势跟踪。有条件的同学可以3D打印机械零件，动手组装，对于理解一台AR的构造很有帮助。电路部分还是建议购买，有第三方提供整套电路板。由于这个项目是开源的，有机械，电路，和软件的全部细节，可以随意改进。所以如果希望实现一个AR头显原型，North Star是一个很好的起点。

![Project North Star](/images/blog/ARVR/PolarisAR.jpg)

### 特点 
* 硬件部分，Project North Star不依托于手机，除了计算还是在PC上之外，具备一体式AR眼镜的大部分细节。 全套机械，硬件自己做下来价格也不算贵，大约3000-4000。主要是显示组件和Leap Motion传感器较贵。 也可以在第三方购买部分或者全部组件： [https://www.smart-prototyping.com/AR-VR-MR-XR](https://www.smart-prototyping.com/AR-VR-MR-XR)  
* 软件部分，只支持Windows和Unity。也就是这个项目只能接PC主机运行，是纯粹的开发原型。

* [https://developer.leapmotion.com/northstar/](https://developer.leapmotion.com/northstar/) 
* [https://github.com/leapmotion/ProjectNorthStar](https://github.com/leapmotion/ProjectNorthStar) 

## Intel RealSense 
RealSense是Intel的计算机视觉传感器品牌。RealSens提供了各种追踪和深度传感器的工业级解决方案。在开发AR项目原型时需要的追踪和深度的解决方案基本可以从RealSense获得。Intel的软件支持很好，覆盖平台很全。 

![Intel RealSense](/images/blog/ARVR/Realsense.jpg)

* [https://www.intel.com/content/www/us/en/architecture-and-technology/realsense-overview.html](https://www.intel.com/content/www/us/en/architecture-and-technology/realsense-overview.html) 
* [https://github.com/IntelRealSense/librealsense](https://github.com/IntelRealSense/librealsense) 

## OpenMV
OpenMV项目提供了一款低价，可扩展，支持Python的机器视觉模块。OpenMV项目代码可以在github上获得全部获得。OpenMV类似Arduino，Raspberry Pi，只不过主要面向计算机视觉领域。OpenMV和RealSense的区别在于，OpenMV基于单目摄像头（即一个摄像头，无法提供深度信息）计算机视觉技术。RealSense是Intel的工业级解决方案，提供的是立体视觉解决方案，传感器都是能提供深度信息的双目，结构光，或者激光的摄像头系统。OpenMV是一个很好的入门计算机视觉的项目。

![OpenMV](/images/blog/ARVR/OpenMV.jpg)

* [https://singtown.com/openmv/](https://singtown.com/openmv/)
* [https://github.com/openmv/openmv](https://github.com/openmv/openmv)

## AR技术的难点 
看完这些项目，可以知道AR眼镜的基础技术主要有：

* **光学显示**：大视场角和高亮度的光学显示组件。
* **SLAM**：环境感知和定位。
* **计算机视觉**：平面识别，物体识别，图像识别。
* **交互技术**：手势识别。

现在实现这些技术都已存在。难点在于一款AR设备如想被普通消费者接受，需要能被普通人舒适地佩戴8小时以上。所以AR眼镜玩家现有的挑战其实是如何在**低功耗**和**小体积**这两个前提下把现有技术整合成一款消费级产品。这并非易事，但技术的发展在超过5到10年这个跨度后，常常超越大部分人的想象。

## 人以群分
建了一个AR技术微信群，面向工程师，主要是为了交流AR技术和项目。如果有兴趣，可以加微信: lincode，备注：彩虹尽头。