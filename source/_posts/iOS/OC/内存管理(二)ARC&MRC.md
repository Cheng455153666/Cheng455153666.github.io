---
title: 内存管理(一)ARC&MRC
date: 2018.11.14 21:33:53
categories: iOS
tags: [iOS]
comment: false
---

## ARC

Q: 什么是ARC？

ARC是由编译器(LLVM)为我们自动插入retain、release之外还需要Runtime协作最终组成了ARC。

ARC中是禁止手动调用retain/release/retainCount/dealloc的。但是在ARC中可以重写对象的dealloc，但是不能在dealloc显示调用`[super dealloc]`。除此之外，ARC中新增了weak、strong属性关键字。

由于ARC中有很大一部分实际上就是由编译器和Runtime自动为我们添加的内存操作，我们还是主要关注MRC中的内存方法的使用情况。

## MRC

iOS中的内存管理，涉及到这么几个方法

* alloc
* **retain**
* **release**
* **retainCount**
* **autorelease**
* dealloc

在ARC下，使用上面的加粗的方法，会引起编译报错。

现在我们就来分析一下这些方法的内部实现原理：

#### alloc的实现

```Objective-C
static ALWAYS_INLINE id
callAlloc(Class cls, bool checkNil, bool allocWithZone=false)
{
    if (slowpath(checkNil && !cls)) return nil;

#if __OBJC2__
    if (fastpath(!cls->ISA()->hasCustomAWZ())) {
        // No alloc/allocWithZone implementation. Go straight to the allocator.
        // fixme store hasCustomAWZ in the non-meta class and 
        // add it to canAllocFast's summary
        if (fastpath(cls->canAllocFast())) {
            // No ctors, raw isa, etc. Go straight to the metal.
            bool dtor = cls->hasCxxDtor();
            id obj = (id)calloc(1, cls->bits.fastInstanceSize());
            if (slowpath(!obj)) return callBadAllocHandler(cls);
            obj->initInstanceIsa(cls, dtor);
            return obj;
        }
        else {
            // Has ctor or raw isa or something. Use the slower path.
            id obj = class_createInstance(cls, 0);
            if (slowpath(!obj)) return callBadAllocHandler(cls);
            return obj;
        }
    }
#endif

    // No shortcuts available.
    if (allocWithZone) return [cls allocWithZone:nil];
    return [cls alloc];
}
```

我们在源码中可以看到，alloc经过一系列的封装调用，最终调用到了`calloc`这个C函数方法。此时并没有设置引用计数为1。但是我们使用`retainCount`获取引用计数时，为1，这是为什么？我们会在retainCount中说明

#### retain的实现

```Objective-C
__attribute__((aligned(16)))
id 
objc_retain(id obj)
{
    if (!obj) return obj;
    if (obj->isTaggedPointer()) return obj;
    return obj->retain();
}
```
我们可以看到，会先判断对象是否是简单类型。是就直接返回，否，则调用obj->retain()。再看看obj->retain()的方法实现。

``` Objective-C
id
objc_object::sidetable_retain()
{
#if SUPPORT_NONPOINTER_ISA
    assert(!isa.nonpointer);
#endif
	// 通过对象本身找到对象是在引用计数表中的哪一张表中
    SideTable& table = SideTables()[this];
    
    table.lock();
    // 在SideTable中获取对象本身的引用计数值
    size_t& refcntStorage = table.refcnts[this];
    if (! (refcntStorage & SIDE_TABLE_RC_PINNED)) {
    	/* 对引用计数值进行+1操作，我们可以看看这个宏定义
    		#define SIDE_TABLE_RC_ONE (1UL<<2) // MSB-ward of deallocating bit
    		这就是之前说到的，右移2位+1的操作
    	*/
        refcntStorage += SIDE_TABLE_RC_ONE;
    }
    table.unlock();

    return (id)this;
}

```

#### release的实现原理

```Objective-C
// rdar://20206767
// return uintptr_t instead of bool so that the various raw-isa 
// -release paths all return zero in eax
uintptr_t
objc_object::sidetable_release(bool performDealloc)
{
#if SUPPORT_NONPOINTER_ISA
    assert(!isa.nonpointer);
#endif
	// 通过Hash查找找到对象所在的SideTable表。
    SideTable& table = SideTables()[this];

    bool do_dealloc = false;

    table.lock();
    // 找到对应的引用计数表
    RefcountMap::iterator it = table.refcnts.find(this);
    if (it == table.refcnts.end()) {
        do_dealloc = true;
        table.refcnts[this] = SIDE_TABLE_DEALLOCATING;
    } else if (it->second < SIDE_TABLE_DEALLOCATING) {
        // SIDE_TABLE_WEAKLY_REFERENCED may be set. Don't change it.
        do_dealloc = true;
        it->second |= SIDE_TABLE_DEALLOCATING;
    } else if (! (it->second & SIDE_TABLE_RC_PINNED)) {
    	// 和retain相同，也是偏移，然后-1
        it->second -= SIDE_TABLE_RC_ONE;
    }
    table.unlock();
    if (do_dealloc  &&  performDealloc) {
        ((void(*)(objc_object *, SEL))objc_msgSend)(this, SEL_dealloc);
    }
    return do_dealloc;
}
```

#### retainCount的实现原理

```Objective-C
uintptr_t
objc_object::sidetable_retainCount()
{
	// 找到对应的引用计数表
    SideTable& table = SideTables()[this];
	// 先声明一个局部变量
    size_t refcnt_result = 1;
    
    table.lock();
    // 查找对象的引用计数表
    RefcountMap::iterator it = table.refcnts.find(this);
    if (it != table.refcnts.end()) {
        // this is valid for SIDE_TABLE_RC_PINNED too
        // 如果找到了就偏移然后加上上面声明的局部变量
        refcnt_result += it->second >> SIDE_TABLE_RC_SHIFT;
    }
    table.unlock();
    return refcnt_result;
}
```

#### dealloc的实现原理

```Objective-C
inline void
objc_object::rootDealloc()
{
    if (isTaggedPointer()) return;  // fixme necessary?

    if (fastpath(isa.nonpointer  &&  // 是否是NONPOINTER_ISA
                 !isa.weakly_referenced  &&  // 挡墙指针是否有weak指针指向
                 !isa.has_assoc  &&  // 是否有关联对象
                 !isa.has_cxx_dtor  &&  // 是否使用到了C++或ARC相关的内容
                 !isa.has_sidetable_rc))    // 是否有引用计数表
    {
        /* 可以直接释放的条件为
            1. 是NONPOINTER_ISA
            2. 没有弱引用指向
            3. 没有关联对象
            4. 没使用C++相关内容
            5. 没有采用sidetable来存储引用计数
         */
        assert(!sidetable_present());
        free(this);
    } 
    else {  // 上面的条件不成立，就要使用这个函数来释放上面用到的资源
        object_dispose((id)this);
    }
}
```

我们再看看`object_dispose`函数内部做了什么

```Objective-C
// 这里是object_dispose方法
id 
object_dispose(id obj)
{
    if (!obj) return nil;

    objc_destructInstance(obj);    
    free(obj);

    return nil;
}

// 这里是object_dispose中调用的objc_destructInstance方法
void *objc_destructInstance(id obj) 
{
    if (obj) {
        // Read all of the flags at once for performance.
        bool cxx = obj->hasCxxDtor();	// 判断是否使用了C++或ARC的内容
        bool assoc = obj->hasAssociatedObjects(); // 判断是否使用了关联对象

        // This order is important.
        if (cxx) object_cxxDestruct(obj); // 调用释放相关资源的函数
        if (assoc) _object_remove_assocations(obj); // 移除关联对象，也就是说，对象的Dealloc中，会自动帮我们在对象被释放的时候清楚对象的关联对象，而不用我们手动移除
        obj->clearDeallocating();
    }

    return obj;
}

// 这里是objc_destructInstance中调用的clearDeallocating方法
inline void 
objc_object::clearDeallocating()
{
    if (slowpath(!isa.nonpointer)) {	// 判断是否是NONPOINTER_ISA
        // Slow path for raw pointer isa.
        sidetable_clearDeallocating();
    }
    else if (slowpath(isa.weakly_referenced  ||  isa.has_sidetable_rc)) {
        // Slow path for non-pointer isa with weak refs and/or side table data.
        clearDeallocating_slow();
    }

    assert(!sidetable_present());
}

void 
objc_object::sidetable_clearDeallocating()
{
    SideTable& table = SideTables()[this];

    // clear any weak table items
    // clear extra retain count and deallocating bit
    // (fixme warn or abort if extra retain count == 0 ?)
    table.lock();
    RefcountMap::iterator it = table.refcnts.find(this);
    if (it != table.refcnts.end()) {
        if (it->second & SIDE_TABLE_WEAKLY_REFERENCED) {
	        // 将指向该对象的弱引用指针置为nil
            weak_clear_no_lock(&table.weak_table, (id)this);
        }
        // table引用计数擦除(从引用计数表中擦除该对象引用计数)
        table.refcnts.erase(it);
    }
    table.unlock();
}

NEVER_INLINE void
objc_object::clearDeallocating_slow()
{
    assert(isa.nonpointer  &&  (isa.weakly_referenced || isa.has_sidetable_rc));

    SideTable& table = SideTables()[this];
    table.lock();
    if (isa.weakly_referenced) {
    	// 将指向该对象的弱引用指针置为nil
        weak_clear_no_lock(&table.weak_table, (id)this);
    }
    if (isa.has_sidetable_rc) {
        // table引用计数擦除(从引用计数表中擦除该对象引用计数)
        table.refcnts.erase(this);
    }
    table.unlock();
}


```
