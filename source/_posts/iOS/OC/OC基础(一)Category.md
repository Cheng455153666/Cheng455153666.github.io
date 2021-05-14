---
title: OC基础(一)Category
date: 2018.10.27 16:12:36
categories: iOS
tags: [iOS]
comment: false
---

Category是Objctive-C 2.0之后所添加的语言特性，可用于为类添加方法(实例方法，类方法，协议，属性)。我们通常使用Category做什么呢？

* 声明私有方法
* 分解体积庞大的文件
* 把Framework私有方法公开
* 模拟多继承

分类这种特性对于纯动态语言来说，不算什么，但是对于C、C++这种编译期决定一切的语言来说，还是非常有用的。

先来说说分类的特性：

* 运行时决议。当我们写好分类后，并没有立刻将写好的分类的方法添加到对应的宿主类中。而是在运行时通过Runtime，在运行时才添加到对应的宿主类上。
* 可以为系统类添加分类。
* 可以为类添加方法(实例方法，类方法，协议)，但是不能添加成员变量(因为在运行时，内存布局就已经确定了，如果添加成员变量，会破坏类的内部结构)
* 可以为类添加属性，这里要注意的是，和我们在类中正常定义属性不同，我们在分类中添加属性，只是生成了对应的set/get方法。并没添加成员变量。

关于运行时决议这点，就不得不说说分类和Extension的区别了。我们通常用Extension来隐藏类的私有信息。

* Extension只是一个声明文件(.h，而Category有声明[.h]和实现[.m])
* Extension是编译时决议的，伴随类的产生而产生，随之消亡而消亡
* 只能为已知的类添加Extension
* Extension可以添加成员变量

### Category编译后的源码分析
到这里，大致说明了分类的特性。但是需要了解其工作原理，还需要看objc的源码。我们打开源码，然后找到分类的结构体定义:

```Objective-C
struct category_t {
    const char *name;   // 分类名(小括号里写的名字)
    classref_t cls;     // 宿主对象 实际上是 class 类型 编译期间这个值是不会有的，在app被runtime加载时才会根据name对应到类对象
    struct method_list_t *instanceMethods;  // 实例方法列表
    struct method_list_t *classMethods;     // 类方法列表
    struct protocol_list_t *protocols;      // 协议列表
    struct property_list_t *instanceProperties; // 属性列表(不过这个property不会@synthesize成员变量，一般有需求添加成员变量属性时会采用objc_setAssociatedObject和objc_getAssociatedObject方法绑定方法绑定，但这种方法生成的与一个普通的成员变量完全是两码事。)
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;   // 类属性列表（@property (class,nonatomic, strong) NSString *name; 然后还需要现实+的getter setter方法，然后就可以用.号使用
	// 这两个是用来筛选方法的，不用关注
    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
```

看到了分类的内部结构，可以很清楚的看到，分类能够为类添加哪些东西。

我们在看看编译器是如何帮我们编译我们创建的分类的

定义一个`Person`类，其中不做任何的事情。然后定义一个分类`Person+MyCat`。添加一个`run`方法

```Objective-C
#import "Person+MyCat.h"

@implementation Person (MyCat)

- (void)run {
    NSLog(@"Person run ...");
}

@end
```

我们使用【clang -rewrite-objc -fobjc-arc `文件名.m`】，编译器命令来编译我们的分类文件`Person+MyCat`，会得到一个.cpp文件。其中大多数对我们要研究的分类是无用的，我们只要关注最下方的几百行代码即可。

这里我们截取对我们探究有用的代码:

```Objective-C
/* @implementation Person (MyCat) 这里是我们在分类中声明的方法的函数实现
   我们可以先看这个方法的命名_I(表示这是一个实例方法)_Person(宿主类名)_MyCat(分类名)_run(方法名)
   其中传入2个参数，Person * self(宿主对象) 和 SEL _cmd(方法选择器)
   那么既然函数为我们定义好了，这个函数是在哪里调用的，我们通过方法名找到调用位置
 */
static void _I_Person_MyCat_run(Person * self, SEL _cmd) {
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_9w_zv4vj4y51q1fjd_d9402351m0000gn_T_Person_MyCat_ba0f5f_mi_0);
}

/*
 我们定位到了这里。这里又是一个结构体 _OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_MyCat 这个名称就不再做解释了
 我们关注调用我们函数的那一行代码
 {{(struct objc_selector *)"run", "v16@0:8", (void *)_I_Person_MyCat_run}}
 来理解一下，为什么函数被包装成了这样
 (struct objc_selector *)"run": 这里将"run"显示转换成了一个方法选择器指针
 "v16@0:8": 这涉及到苹果的type encoding技术，相关的技术，会在之后的章节说明
 (void *)_I_Person_MyCat_run: 这里是我们的函数
 
 那么，又是谁调用到了_OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_MyCat？
 */
static struct /*_method_list_t*/ {	// 这里用一个method_t数组来存储方法
	unsigned int entsize;  // sizeof(struct _objc_method) 方法大小
	unsigned int method_count;  // 方法数量
	struct _objc_method method_list[1]; // 方法本身
} _OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_MyCat __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_objc_method),
	1,
	{{(struct objc_selector *)"run", "v16@0:8", (void *)_I_Person_MyCat_run}}
};

// 上面的结构体在这里被调用了 _category_t 实际上就是category_t的结构体 我们可以按照属性对应一下
// 要注意的是，为了方便查看各个不同情况的方法，协议所添加的位置，又在分类中添加了一个静态方法和一个属性以及一个协议
static struct _category_t _OBJC_$_CATEGORY_Person_$_MyCat __attribute__ ((used, section ("__DATA,__objc_const"))) = 
{
	"Person", // 宿主类名
	0, // &OBJC_CLASS_$_Person,
	(const struct _method_list_t *)&_OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_MyCat,// 实例方法名
	(const struct _method_list_t *)&_OBJC_$_CATEGORY_CLASS_METHODS_Person_$_MyCat,// 类方法名
	(const struct _protocol_list_t *)&_OBJC_CATEGORY_PROTOCOLS_$_Person_$_MyCat,// 协议
	(const struct _prop_list_t *)&_OBJC_$_PROP_LIST_Person_$_MyCat,// 属性
};
```

通过上面的源码 我们可以看出，分类就是将方法封装在一个`method_list_t`的数组当中，其中的每个对象都是`method_t`，`method_t`仅仅是一个封装了函数选择器，函数返回值和参数，函数实现的结构体。

```Objective-C
struct method_t {
    SEL name;   // 方法选择器
    const char *types;  // 函数返回值和参数
    IMP imp;    // 函数实现地址

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};
```

在回到编译的源码，我们可以看到，`_OBJC_$_CATEGORY_Person_$_MyCat`最后被放在了DATA段下的`objc_catlist`中，其中保存了一个大小为1的`category_t`数组。如果有多个分类，就是多个`category_t`啦！

```Objective-C
static struct _category_t *L_OBJC_LABEL_CATEGORY_$ [1] __attribute__((used, section ("__DATA, __objc_catlist,regular,no_dead_strip")))= {
	&_OBJC_$_CATEGORY_Person_$_MyCat,
};
```

大致结构如下：

![Xnip2018-10-27_14-57-40.png](https://upload-images.jianshu.io/upload_images/8037794-037995605634bdf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### Category是如何加载的？

前面说到了，Category是在运行时加载的，所以，到底是在哪一步被加载的？先来看看Category的主要的几个调用方法。

1. 当程序启动后，对象在运行时会调用`_objc_init`方法
2. 在初始化过程中会调用`map_2_images`函数，这里函数名中的images表示的是镜像
3. 之后又会调用`map_images_nolock`
4. 再接下来会调用`_read_images`，读取镜像，加载一些可执行文件到内存当中，我们在上面说到编译时被添加的catlist在这里就用到了
5. 最后调用`remethodizeClass`，真正分类的加载就在这一步，我们就从这一步开始，分析分类的加载流程

```Objective-C
/***********************************************************************
* remethodizeClass
* Attach outstanding categories to an existing class.
* Fixes up cls's method list, protocol list, and property list.
* Updates method caches for cls and its subclasses.
* Locking: runtimeLock must be held by the caller
**********************************************************************/
static void remethodizeClass(Class cls)
{
    category_list *cats;
    bool isMeta;

    runtimeLock.assertWriting();
    
    /*
     这里我们按照被添加的方法是实例方法进行说明，即 isMeta = NO
     关于Meta是什么，在后面的章节中，会做讲解
     */
    isMeta = cls->isMetaClass();

    // Re-methodizing: check for more categories
    // 获取cls中未完成整合的所有分类 获取的是分类的列表
    if ((cats = unattachedCategoriesForClass(cls, false/*not realizing*/))) {
        if (PrintConnecting) {
            _objc_inform("CLASS: attaching categories to class '%s' %s", 
                         cls->nameForLogging(), isMeta ? "(meta)" : "");
        }
        // 获取到了 就将分类cats拼接到cls上
        attachCategories(cls, cats, true /*flush caches*/);        
        free(cats);
    }
}

// 再来看看attachCategories这个方法做什么？
// Attach method lists and properties and protocols from categories to a class.
// Assumes the categories in cats are all loaded and sorted by load order, 
// oldest categories first.
static void 
attachCategories(Class cls, category_list *cats, bool flush_caches)
{
    // 这里做了一些空值判断
    if (!cats) return;
    if (PrintReplacedMethods) printReplacements(cls, cats); // Debug调试的无关内容
    // 依照上面的前提 isMeta = NO
    bool isMeta = cls->isMetaClass();

    
    /* 声明了一些变量。这些变量的类型都是二位数组，以method_list_t为例，其结构如下
        [[method_t, method_t, ...], [method_t], [method_t, method_t, method_t], ...]
     */
    
    // fixme rearrange to remove these intermediate allocations
    method_list_t **mlists = (method_list_t **)
        malloc(cats->count * sizeof(*mlists));
    property_list_t **proplists = (property_list_t **)
        malloc(cats->count * sizeof(*proplists));
    protocol_list_t **protolists = (protocol_list_t **)
        malloc(cats->count * sizeof(*protolists));

    // Count backwards through cats to get newest categories first
    // 通过cats向后计数以获得最新的类别
    int mcount = 0; // 方法数量
    int propcount = 0;  // 属性数量
    int protocount = 0; // 协议数量
    int i = cats->count;    // 宿主类分类的总数
    bool fromBundle = NO;
    while (i--) {   // 这里做了一个倒叙遍历，即 最先访问最后编译的分类
        auto& entry = cats->list[i];    // 获取一个分类

        method_list_t *mlist = entry.cat->methodsForMeta(isMeta); // 获取实例方法列表(因为我们假定isMeta = NO)
        if (mlist) {// 如果方法列表存在
            mlists[mcount++] = mlist;   // 添加实例方法列表到二维数组的第mcount位
            fromBundle |= entry.hi->isBundle();
        }
        // 下面的属性 协议 也一样 顺次遍历 最终的添加结果就是上面我们说到的method_list_t的内部结构(二维数组)
        property_list_t *proplist = 
            entry.cat->propertiesForMeta(isMeta, entry.hi);
        if (proplist) {
            proplists[propcount++] = proplist;
        }

        protocol_list_t *protolist = entry.cat->protocols;
        if (protolist) {
            protolists[protocount++] = protolist;
        }
    }
    // 这里获取了宿主类当中的rw数据，这个rw包含了宿主类的信息(在runtime中会讲到这个rw)
    auto rw = cls->data();

    // 如果分类中有一些关于内存管理相关的方法的情况下 做一些特殊处理
    prepareMethodLists(cls, mlists, mcount, NO, fromBundle);
    
    /******
     这里就是关键方法了，将分类方法，将mcount个元素的二维数组mlists 拼接到宿主类的对于方法列表中。
     也就是说，在这行代码执行后，分类才被真正的添加到宿主类中。
     ******/
    rw->methods.attachLists(mlists, mcount);
    free(mlists);
    if (flush_caches  &&  mcount > 0) flushCaches(cls);

    rw->properties.attachLists(proplists, propcount);
    free(proplists);

    rw->protocols.attachLists(protolists, protocount);
    free(protolists);
}

// 我们再看看attachLists内部的具体实现
/*
 addedLists 是传递过来的二位数组，我们假定以下3个分类A,B,C
 
 [[method_t, method_t, ...], [method_t], [method_t, method_t, method_t], ...]
 |____分类A中的方法列表(A)___|  |____B___|  |________________C_________________|
 
 addedCount = 3
 */
void attachLists(List* const * addedLists, uint32_t addedCount) {
    // 空值判断
    if (addedCount == 0) return;

    /*
     这里我们只分析第一个 if 中的判断，因为下面两个else中的判断是列表是采用array还是list_t这种结构而做的不同处理，
     对我们分析分类的实现逻辑不产生影响
     */
    
    if (hasArray()) {
        // many lists -> many lists
        // 获取列表中原有数据个数 如果原本有2个元素 需要被添加的有3个元素
        uint32_t oldCount = array()->count;
        // 拼接之后的元素总个数  2 + 3
        uint32_t newCount = oldCount + addedCount;
        // 按照计算的总个数，重新分配内存
        setArray((array_t *)realloc(array(), array_t::byteSize(newCount)));
        // 重新设置元素总数
        array()->count = newCount;
        /* 内存移动
         [[],[],[],[原有的第一个元素],[原有的第二个元素]]
         */
        memmove(array()->lists + addedCount, array()->lists, 
                oldCount * sizeof(array()->lists[0]));
        /* 内存拷贝
         [
            addedLists中的第一个元素   ---->  [A],
            addedLists中的第二个元素   ---->  [B],
            addedLists中的第三个元素   ---->  [C],
            [原有的第一个元素],
            [原有的第二个元素],
         ]
         */
        memcpy(array()->lists, addedLists, 
               addedCount * sizeof(array()->lists[0]));
    }
    else if (!list  &&  addedCount == 1) {
        // 0 lists -> 1 list
        list = addedLists[0];
    } 
    else {
        // 1 list -> many lists
        List* oldList = list;
        uint32_t oldCount = oldList ? 1 : 0;
        uint32_t newCount = oldCount + addedCount;
        setArray((array_t *)malloc(array_t::byteSize(newCount)));
        array()->count = newCount;
        if (oldList) array()->lists[addedCount] = oldList;
        memcpy(array()->lists, addedLists, 
               addedCount * sizeof(array()->lists[0]));
    }
}
```

到这里，我们对分类的加载情况，就有一个大致的理解了。总结一下：

* 分类添加的方法可以`覆盖`原类方法
* 同名分类方法谁能生效取决于编译顺序

同时，我们也可以解释下面几个问题：

Q: 同一对象的不同分类拥有相同的方法，在程序执行时，会执行哪个？

这个我们在上面的源码中可以看到，最后被加载的方法，会`覆盖`掉之前的方法。(这里要说明的是，这个覆盖，可不是替换，而且在方法查找的过程中，因为分类的方法被加到了最前面，因此，在方法选择器找到响应方法后，就直接返回了，不会再执行后面的方法。)

Q: 分类为什么会覆盖掉父类的方法实现？

我们在上面的题中也说到了，因为分类的方法列表被插入到了原方法之前，所以分类方法被优先找到了。换句话说，实际上父类的方法一直都存在，只是因为查找顺序的关系，一直不会被执行而已。

---

到这里，分类的基本内容就讲述完了，再回过头想想，我们如何通过分类来添加成员变量呢？