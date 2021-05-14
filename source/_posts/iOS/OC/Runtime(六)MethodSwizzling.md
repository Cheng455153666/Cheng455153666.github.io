---
title: (六)MethodSwizzling
date: 2018.11.12 20:31:11
categories: iOS
tags: [iOS]
comment: false
---

Q: 什么是Method-Swizzling?

实际上就是交换两个方法的实现!

![Xnip2018-10-24_21-38-30.png](https://upload-images.jianshu.io/upload_images/8037794-8262e2afc4d95859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实际上实现起来很简单

```Objective-C

// 获取self的方法method1,method2
Method m1 = class_getInstanceMethod(self, @selector(method1));
Method m2 = class_getInstanceMethod(self, @selector(method2));
// 交换实现
method_exchangeImplementations(m1, m2);

```
这样两个方法的实现就被交换了。