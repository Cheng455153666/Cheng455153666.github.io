---
title: Runtime(五)消息转发流程
date: 2018.11.11 19:51:12
categories: iOS
tags: [iOS]
comment: false
---

我们先来看看实例方法的消息转发流程

![Xnip2018-10-24_17-42-07.png](https://upload-images.jianshu.io/upload_images/8037794-7ae078743cdd5dc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里来说明一下流程

1. 先调用`resolveInstanceMethod:`这个类方法，告诉系统我们是否需要通过消息转发来处理此方法。(如果是类方法，调用`resolveClassMethod:`这个方法)，此方法的参数是方法选择器。
2. 当我们返回YES，则相当于告诉系统，当前方法已处理，也就结束了消息转发流程。
3. 如果返回NO，系统会再去找`forwardingTargetForSelector:`这个对象方法，这个对象的返回值就是告诉系统，这个方法(SEL)，应当由哪个对象处理，系统会将这个方法转发给返回的这个转发目标，然后结束消息转发流程。
4. 如果在`forwardingTargetForSelector:`方法我们返回nil，系统会再寻找`methodSignatureForSelector:`方法，这也是处理转发消息的最后一次机会。这个方法的返回值是`NSMethodSignature`，这个对象是对方法选择器的返回值类型以及参数个数和类型的一个封装。
5. 当我们在`forwardingTargetForSelector:`返回了一个方法签名(NSMethodSignature)，系统会调用`forwardInvocation:`方法。我们在这个方法中处理消息。到这里，消息转发流程就结束了。
6. 但是如果我们在`methodSignatureForSelector:`返回nil，

```Objective-C
[NSMethodSignature signatureWithObjCTypes:"v@:"];
// 这是我们之前说到的type encoding
```