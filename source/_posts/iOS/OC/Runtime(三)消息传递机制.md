---
title: Runtime(三)消息传递机制
date: 2018.11.09 22:01:22
categories: iOS
tags: [iOS]
comment: false
---

我们知道在OC中，所有的方法调用最终都会转换成`objc_msgSend`形式的方法调用。如下图:

![Xnip2018-10-24_16-31-38.png](https://upload-images.jianshu.io/upload_images/8037794-c2c2d0dacb1eecac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而对于调用父类的方法，用的是另一个方法`objc_msgSendSuper`。

![Xnip2018-10-24_16-40-01.png](https://upload-images.jianshu.io/upload_images/8037794-a50c727bb048db12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们再来看看super的结构体

```Objective-c
/// Specifies the superclass of an instance.
/// 指定实例的超类。
struct objc_super {
    /// Specifies an instance of a class.
    /// 指定类的实例
    __unsafe_unretained _Nonnull id receiver;

    /// Specifies the particular superclass of the instance to message.
#if !defined(__cplusplus)  &&  !__OBJC2__
    /* For compatibility with old objc-runtime.h header */
    __unsafe_unretained _Nonnull Class class;
#else
    __unsafe_unretained _Nonnull Class super_class;
#endif
    /* super_class is the first class to search */
};
```

我们重点关注`__unsafe_unretained _Nonnull id receiver;`这个对象，super是一个编译器关键字，经过编译器编译后，会被解析成objc_super类型的结构体指针，而其中的receiver成员变量就指向当前的对象。

我们再回来来看看方法传递的流程

![Xnip2018-10-24_16-59-46.png](https://upload-images.jianshu.io/upload_images/8037794-0d5288ef17a4b9bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们会在方法缓存章节中详细说明系统是如何进行方法缓存的