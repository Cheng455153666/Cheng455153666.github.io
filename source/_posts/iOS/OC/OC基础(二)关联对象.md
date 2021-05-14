---
title: OC基础(二)关联对象
date: 2018.11.01 10:11:32
categories: iOS
tags: [iOS]
comment: false
---

在分类的源码中，我们并没有看到分类有能够添加成员变量能力，但是我们可以通过关联对象的方式来达到在分类中为对象添加成员变量的效果。 我们先来看看有关关联对象的几个方法：

```Objective-C
// 根据key 从object对象中获取对应的关联值 并作为返回值返回
id objc_getAssociatedObject(id object, const void *key)

// 首先设置value值 然后object通过指定的key 通过指定的策略policy和value建立映射关系
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)

// 移除object的所有关联对象
void objc_removeAssociatedObjects(id object)
```

结合源码，我们是可以知道分类中是没有存储成员变量的能力的。那么，我们在分类中添加的成员变量都被添加到哪里去了？

我们在分类中添加成员变量是使用`关联对象`的技术。而关联对象可以达到添加成员变量的效果(注意，是效果，但是成员变量并没有被实际添加到类中的！)。

### 关联对象

我们都知道，类的定义是在编译期间就已经在内存中固定了的，也是因为这个原因，我们是无法通过任何方法修改到内存中的固定区域的。因此，为了让我们能够在类中`为对象添加`成员变量。苹果让我们能够使用关联对象的方式。为类添加需要的成员变量，其本质很简单，就是在程序运行的过程中，创建了一个对象(AssociationsManager)，这个对象负责维护一个全局容器Map(AssociationsHashMap)，而这个Map就通过对象指针(DISGUISE(obj))又维护了子Map(ObjectAssociationMap)，这个子Map通过我们传入的key值，存储着我们为类`添加`的成员变量及其关联策略。如下图：

![Xnip2018-10-18_23-03-52.png](https://upload-images.jianshu.io/upload_images/8037794-699bcc568063252c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们再来通过源码，分析一下关联对象的具体实现。

这里我们主要分析`void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)`的实现：

```Objective-C
// 我们可以看到 objc_setAssociatedObject 实际上就是透传了_object_set_associative_reference方法，我们直接看_object_set_associative_reference方法的实现

/*
 object: 要关联的对象
 key: 关联的key
 value: 关联的z值
 policy: 关联策略
 */
void _object_set_associative_reference(id object, void *key, id value, uintptr_t policy) {
    // retain the new value (if any) outside the lock.
    ObjcAssociation old_association(0, nil);
    /* 按照传入的策略policy，对值进行包装
     static id acquireValue(id value, uintptr_t policy) {
         switch (policy & 0xFF) {
         case OBJC_ASSOCIATION_SETTER_RETAIN:   // retain 包装
             return ((id(*)(id, SEL))objc_msgSend)(value, SEL_retain);
         case OBJC_ASSOCIATION_SETTER_COPY:     // copy 包装
             return ((id(*)(id, SEL))objc_msgSend)(value, SEL_copy);
         }
         return value;
     }
     */
    id new_value = value ? acquireValue(value, policy) : nil;
    {
        // 声明了一个 AssociationsManager 这是一个全局对象
        AssociationsManager manager;
        // 被上面的 AssociationsManager 管理的全局Hashmap容器 可以理解为一个字典
        AssociationsHashMap &associations(manager.associations());
        // 根据对象的内存地址 计算出一个指针地址值 来作为某对象的唯一标识
        disguised_ptr_t disguised_object = DISGUISE(object);
        if (new_value) {    // 准备被关联的对象有值
            // break any existing association.
            // 查找对象在全局容器中的map 如果这个对象不是第一次关联对象，那这个容器在首次关联对象就被创建好了
            AssociationsHashMap::iterator i = associations.find(disguised_object);
            if (i != associations.end()) {  // 找到了对象对应Map
                // secondary table exists
                // 取到对象关联的ObjectAssociationMap
                ObjectAssociationMap *refs = i->second;
                // 通过key找到对应的 ObjcAssociation(这个结构封装了关联策略和值)
                ObjectAssociationMap::iterator j = refs->find(key);
                if (j != refs->end()) { // 找到了key相同的，则替换值
                    old_association = j->second;
                    j->second = ObjcAssociation(policy, new_value);
                } else {    // 没找到则按key 关联 ObjcAssociation
                    (*refs)[key] = ObjcAssociation(policy, new_value);
                }
            } else {    // 没找到对象对应Map
                // create the new association (first time).
                // 创建一个 ObjectAssociationMap 对象
                ObjectAssociationMap *refs = new ObjectAssociationMap;
                // 在全局容器中，用对象的key(disguised_object)关联这个新创建的ObjectAssociationMap
                associations[disguised_object] = refs;
                // 通过传入的key 关联 ObjcAssociation(这个结构封装了关联策略和值)
                (*refs)[key] = ObjcAssociation(policy, new_value);
                // 设置对象已持有关联对象的标识位
                object->setHasAssociatedObjects();
            }
        } else {    // 如果没有值，系统就把这次操作当成是按照Key移除对象某个值的操作
            // setting the association to nil breaks the association.
            // 查找对象在全局容器中的map
            AssociationsHashMap::iterator i = associations.find(disguised_object);
            if (i !=  associations.end()) { // 找到了map， 如果没找到就什么也不做
                // 找到对象对应的ObjectAssociationMap
                ObjectAssociationMap *refs = i->second;
                // 找到key在ObjectAssociationMap对应的值
                ObjectAssociationMap::iterator j = refs->find(key);
                if (j != refs->end()) { // 如果值存在， 如果不存在就什么都不做
                    // 找到 key 关联的 ObjcAssociation
                    old_association = j->second;
                    // 擦除关联
                    refs->erase(j);
                }
            }
        }
    }
    // release the old value (outside of the lock).
    if (old_association.hasValue()) ReleaseValue()(old_association);
}

```

再看看关联对象的本质

```Json
{
	"0x4827298742": {	// 被关联的对象(地址)
		@selector(text): {	// 我们传入的key
			"value": "Hello",	// 被关联的值
			"policy": "retain"	// 关联策略
		},
		@selector(title): {
			"value": "a object",
			"policy": "copy"
		}
	},
	"0x3413513412": {
		@selector(backgroundColor): {
			"value": "0xff8205",
			"policy": "retain"
		}
	}
}
```

到这里，关联对象的实现大家基本都了解了吧~