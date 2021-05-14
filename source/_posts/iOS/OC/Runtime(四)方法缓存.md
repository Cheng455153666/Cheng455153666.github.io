---
title: Runtime(四)方法缓存
date: 2018.11.10 12:31:55
categories: iOS
tags: [iOS]
comment: false
---

方法缓存的查找流程，实际上就是按照给定的SEL，在方法缓存列表中找到对应的bucket_t中的IMP。对应的流程就是：

![Xnip2018-10-24_17-17-09.png](https://upload-images.jianshu.io/upload_images/8037794-ad170e7513262078.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们通过给定的方法映射出方法在bucket_t中的位置，这个查找的方式，实际上就是哈希查找。其查找方式，就是根据给定的值，与对应的mask(cache_t中的一个成员变量)做位与操作然后计算，取得的一个值。然后在bucket_t中找到对应的IMP，返回给调用方就可以了。

##### 在当前类中查找方法分两种情况：

* 对于`已排序好`的列表，采用`二分查找`算法查找方法对应执行函数。
* 对于`没有排序`的列表，采用`一般遍历`查找方法对应的执行函数。

##### 父类逐级查找流程

![Xnip2018-10-24_17-34-38.png](https://upload-images.jianshu.io/upload_images/8037794-1c2b95e7411e65ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 当前类对象先找到父类对象
2. 判断父类对象是否为nil，如果是nil，则结束方法查找
3. 如果父类不为nil，查找父类方法缓存，找到则返回并结束
4. 如果未在父类缓存中找到方法，则在父类方法列表中查找，找到则返回并结束
5. 如果父类方法列表中没有此方法，则重复到1，循环此流程