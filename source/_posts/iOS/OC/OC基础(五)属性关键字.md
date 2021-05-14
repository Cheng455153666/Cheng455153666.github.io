---
title: OC基础(五)属性关键字
date: 2018.11.05 20:11:26
categories: iOS
tags: [iOS]
comment: false
---

属性关键字可以分为那里类？

* 读写权限
	* readonly
	* readwrite(默认)
* 原子性
	* atomic(默认，是线程安全的，仅对赋值，获取保证线程安全。比如，一个数组使用atomic修饰，我们对数组进行赋值和获取是保证线程安全的，但是我们对数组进行添加删除等操作，是不保证线程安全的)
	* nonatomic
* 引用计数
	* retain(MRC)/strong(ARC)
	* assign(基本数据类型，对象类型)/unsafe_unretained(MRC,ARC基本不用)
	* weak
	* copy

下面分别对这些属性关键字做讲解

#### assign

* 主要用来修饰基本数据类型(int, BOOL等)
* 修饰对象类型时，不改变其引用计数
* 会产生悬垂指针(野指针，即：对象释放后，指针依然指向原地址)

#### weak

* 不改变被修饰对象的引用计数
* 在所指对象被释放后自动置为nil

#### copy

关于copy先来看一个常见面试题

```Objective-C
@property (copy) NSMutableArray *array;
```
这种写法会有什么问题？这里主要考察的是深/浅拷贝的理解。关于深/浅拷贝，一句话概括就是。深拷贝会拷贝一份新的内容相同的对象，并将指针指向新的对象的内存地址。而浅拷贝只会新建一个指针，将指针指向被拷贝的对象的内存地址。

我们可以总结一下，深拷贝不会增加被拷贝对象的引用计数，因为没有新指针指向原对象，但是深拷贝产生了新的内存分配。

所以深/浅拷贝主要的区别点在于

* 是否开辟了新的内存空间
* 是否影响了引用计数

下面是对于可变/不可变对象的拷贝是否是深浅拷贝的情况

![Xnip2018-10-20_23-02-37.png](https://upload-images.jianshu.io/upload_images/8037794-1a2817b9696dbe22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


Q: MRC下如何重写retain修饰变量的setter方法？

```Objective-C
@property (nonatomic, retain) id obj;
// set方法中的实现
- (void)setObj:(id)obj {
	if (_obj != obj) {  // 我们思考一下为什么要添加这个不等的条件?
		[_obj release];
		_obj = [obj retain];
	}
}
```
关于这段代码，为什么添加不等的操作，很多人可能是觉得为了防止相同对象赋值，走多次而产生不必要的开销。但是实际上，这个不等的操作是为了防止当对象引用计数为1，当我们对对象release后，对象被释放了，此时我们又对一个已经释放的对象做retain操作，这样就会造成程序崩溃。

这一块的内容主要会在内存管理部分探究