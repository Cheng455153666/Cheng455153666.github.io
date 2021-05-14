---
title: OC基础(三)代理、通知
date: 2018.11.02 12:30:03
categories: iOS
tags: [iOS]
comment: false
---

## 代理(Delegate)

Q: 什么是代理？

* 准确的说代理是一直软件设计模式(代理模式)
* iOS中以**@protocol**的形式体现
* 传递方式是**一对一**

下面简述一下代理的工作流程


![Xnip2018-10-19_11-26-58.png](https://upload-images.jianshu.io/upload_images/8037794-2804daffe25139a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里我们需要知道一下，协议中可以定义哪些内容？协议可以通过`@required`定义必须实现的代理方法，通过`@optional`定义可选方法

* 成员属性
* 方法

在使用代理的时候，存在一个循环引用的问题，当代理，协议，委托都通过强引用形成一个闭环，则会造成内存泄露的问题，此时我们通常会让委托方弱引用指向代理方来避免循环引用：

![Xnip2018-10-19_11-32-18.png](https://upload-images.jianshu.io/upload_images/8037794-992794b9808c792c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# NSNotification

这里先一下通知(NSNotification)的特点

* 使用**观察者模式**来实现的用于跨层传递消息的机制
* 传递方式为**一对多**

通过一幅图看看通知的大致实现机制

![Xnip2018-10-19_11-40-32.png](https://upload-images.jianshu.io/upload_images/8037794-15eaf2319a21c3c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
