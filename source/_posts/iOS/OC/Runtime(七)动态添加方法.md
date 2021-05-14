---
title: Runtime(七)动态添加方法
date: 2018.10.27 16:12:36
categories: iOS
tags: [iOS]
comment: false
---

在说动态添加方法之前，我们先来看一个问题

Q: 使用`performSelector:`可能会遇到什么问题？

这个方法不会检查对象的方法实现，所以可能会Crash。

我们可以在消息传递的时候动态的添加方法。

```Objective-C
void testImp(void) {
	NSLog(@"this is test method!");
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
	/*
	 这个方法的参数：
	 cls: 为哪个类添加方法
	 name: 方法名
	 imp: 方法实现
	 types: 方法返回值和方法参数
	*/
	class_addMethod(self, @selector(aTestMethod), testImp, "v@:");
	return YES;
}
```

# 动态方法解析

Q: 是否使用过`@dynamic`?

当我们声明一个属性，并且在实现中声明为`@dynamic`时，是告诉编译器，这个属性的`get/set`方法在运行时添加，而不是在编译时添加方法的实现。

这里涉及到动态运行时语言和编译时语言的区别

* 动态运行时语言将函数决议推迟到运行时
* 编译时语言在编译期间决议

## Runtime实战

Q: [obj foo]和objc_msgSend()函数之间有什么关系?

Q: runtime如何通过Selector找到对应的IMP地址的？

Q: 能否向编译后的类中增加实例变量？