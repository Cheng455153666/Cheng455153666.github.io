---
title: OC基础(四)KVC、KVO
date: 2018.11.04 18:10:03
categories: iOS
tags: [iOS]
comment: false
---

## KVO

Q: 什么是KVO?

* KVO(key-value observing)，从名称上就可以知道这是一种键值观察的机制
* KVO是OC对**观察者模式**的又一实现
* Apple使用**isa混写技术(isa-swizzling)**来实现KVO

当我们注册一个对象的观察者的时候，也就是调用了系统的`addObserver:forKeyPath:options:context:`方法，来观察一个对象的某个属性，系统会在运行时创建一个新的对象`NSKVONotifying_对象类名`，并将原本指向`对象类`的isa指针指向这个`NSKVONotifying_对象类名`，`NSKVONotifying_对象类名`实际上是`原对象类的子类`，只是我们在这个子类中，重写了需要观察的属性的`set`方法。以此达到观察值变化的目的，如下图所示：

![Xnip2018-10-19_21-50-35.png](https://upload-images.jianshu.io/upload_images/8037794-9e61773e0918c69b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


Q: 那么，为什么系统新添加的set方法就能够监听到值变化呢？

系统为我们实现的set方法中有这两个方法：

* \- (void)willChangeValueForKey:(NSString *)key
* \- (void)didChangeValueForKey:(NSString *)key

这两个方法分将对属性赋值的代码包了起来

```Objective-C
- (void)setValue:(id)obj {
	[self willChangeValueForKey:@"keyPath"];
	// 调用父类的实现， 也即是原类的实现
	[super setValue:obj];
	[self didChangeValueForKey:@"keyPath"];
}
```

Q: 通过KVC设置的value，KVO能否监听到变化？

答案是能！那么为什么会KVC设置的值，会生效呢？实际上就是要因为在赋值时会调用到属性的set方法，所以会使得KVO被触发。

Q: 通过成员变量直接赋值value，KVO能否有效？

答案是不能，系统为我们提供的KVO实际上还是对set方法的覆盖，但我们直接对成员变量赋值是不会走set方法的。但是，我们可以手动的实现KVO，也就是将上面的`willChangeValueForKey:`和`didChangeValueForKey:`按照上面的样子。将设置值的代码包起来即可。

## KVC

Q: 什么是KVC

* KVC是键值编码(key-value coding)的缩写，和键值编码相关的方法
	* \- (id)valueForKey:(NSString *)key，这个方法可以获取和key同名，或者相似名称的属性值，并返回
	* \- (void)setValue:(id)value forKey:(NSString *)key 可以获取和这个key同名或者相似名称的属性赋值

我们现在来看看KVC的获取属性的流程

![Xnip2018-10-19_22-37-59.png](https://upload-images.jianshu.io/upload_images/8037794-cf7c82302df6a1bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


1. 首先会判断我们使用的key(或相似)是否有get方法，如果get方法存在，则直接调用，结束访问流程
2. 如果访问的key的get方法未找到，则会判断是否有同名或相似的实例变量存在，如果存在，则返回
3. 在get方法未找到的过程中，系统给我们提供了一个可屏蔽的开关`+ (BOOL)accessInstanceVariablesDirectly`，如果和key相同或同名的实例变量存在，则返回YES，但是如果我们重写这个方法并返回NO，即使这个同名实例变量存在，也不会被获取到。
4. 如果实例变量不存在，系统会调用当前实例的`valueForUndefinedKey:`方法，然后抛出`NSUndefinedKeyException`异常，结束调用流程。

这里有两个点需要注意：

* 访问器方法(Accessor Method)是否存在的判断规则时什么样的
* 实例变量(Instance var)是否存在的规则时什么样的

#### 访问器方法(Accessor Method)

访问器方法也实现了`相似`的概念，但是方法的命名必须遵循驼峰命名下面这几种是可以被判断为实现了`key`的访问器方法：

* getKey
* key
* isKey

#### 实例变量(Instance ver)判断

下面几种实例变量的名称是可以被识别的

* _key
* _isKey
* key
* isKey

再让我们现在来看看KVC的Set方法的流程

![Xnip2018-10-20_21-48-16.png](https://upload-images.jianshu.io/upload_images/8037794-0628192ae43c224e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


其方法查询流程，与get的流程相同








