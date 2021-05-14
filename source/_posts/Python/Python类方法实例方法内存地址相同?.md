---
title: Python类方法实例方法内存地址相同?
date: 2018.05.24 17:16:58
categories: Python
tags: [Python]
comment: false
---

问题来源 : 什么是绑定到对象的方法、绑定到类的方法、解除绑定的函数、如何定义，如何调用，给谁用？有什么特性?

从代码表现上来说

```python
class Person(object):
    def func01(self):
        print('绑定到对象的方法')
    
    @classmethod
    def func02(cls):
        print('绑定到类的方法')
    
    @staticmethod
    def func03():
        print('非绑定方法')
```

从代码调用上来说

```Python
p1 = Person()
p1.func01()     # 需要通过对象调用
Person.func02() # 通过类对象调用
Person.func03() # 和类对象调用的方式类似
# 这里可以看到绑定到类对象的函数和非绑定函数的调用方式类似
```

以上三种函数对象的打印输出为
绑定到对象的函数: `<bound method Person.func01 of <__main__.Person object at 0x10b4c9ac8>>`
绑定到类的函数: `<bound method Person.func02 of <class '__main__.Person'>>`
非绑定函数: `<function Person.func03 at 0x10b4f0b70>`

```Python
# 这里出现了个问题 关于python方法调用的流程, 实在是不理解
print(id(p1.func01))        # 4578451592
print(id(p2.func01))        # 4578451592

print(id(Person.func02))    # 4578451592
print(id(Person.func02))    # 4578451592
print(id(Person.func03))    # 4578581360
print(id(Person.func03))    # 4578581360

print(Person().func01 == Person.func02) # False
print(id(Person().func01) == id(Person.func02)) # True

```

*针对上面我不理解的问题 这里做出我的理解*

我们先来看看`Person.__dict__`的输出, 在我定义了对象方法, 类方法, 静态方法后, 这些方法都是在Person这个类的`__dict__`中的, 一下是输出(我们只关注这个几个方法, 其他的打印不写出):

```shell
'func01': <function Person.func01 at 0x109bef9d8>, 
'func02': <classmethod object at 0x109bf7b00>, 
'func03': <staticmethod object at 0x109bf7b70>
```

这里可以明确的看出这几个方法是什么方法, 且内存地址是不一致的. 到这里完全是可以理解的, 在我的认知中, 不同名, 不同类型的方法确实不应当拥有相同的地址.所以,在方法存储的环节,我认为这些方法在内存的表现是让我满意的.

那么是什么导致我看到的地址相同?既然存储没问题, 那么可能导致内存地址相同的关键是在方法调用的过程中.

先来看看对象方法和类方法的内存地址

```Python
print(hex(id(Person().func01))) # 0x10c5a6048
print(hex(id(Person.func02)))   # 0x10c5a6048

print(Person.__dict__['func01'])
# <function Person.func01 at 0x109bef9d8>
print(Person.__dict__['func02'])
# <classmethod object at 0x109bf7b00>
```

上面的类方法和对象方法的地址是相同的, 但从下面可以看出他们在类对象的__dict__中依旧是各自的地址, 接下来就是问题的关键了!!!

当我们通过实例化对象的时候, Python编辑器会将类方法的对象重新使用描述器包装一下, 然后存储到一个新的内存空间, 因此我们调用Person.func01这样的方法,实际上调用的并不是dict中的方法对象, 而是包装过的一个副本!简单的说就是, `不论我们通过Person.func01还是Person().func02去调用方法, 在这个调用的过程中, Python内部会对Person.__dict__['func01']和Person.__dict__['func02']做一次拷贝, 所以我们调用的只是方法的副本. 那么,为什么会出现相同的地址呢?我们都知道Python采用的是垃圾回收的机制, 当一个内存, 没有对象对其引用的话, 就会立刻销毁这块内存然后对其复用. 这个问题发生的关键就在于这个复用阶段. 我们可以推测, 如果不销毁这块内存, Python自然就会开辟新的内存空间去创建对象了吧?`我们做个试验.

```Python
# 这里我们要做的就是防止编译器对func01方法立刻销毁, 我们将他加入列表或者用一个变量指向都行
p1_func01 = Person().func01 # 这里用一个变量指向了一下

print(hex(id(p1_func01)))   # 0x102eb20c8
print(hex(id(Person.func02)))   # 0x102eacdc8
```

现在我们就看到了我们期望的结果了, 对象方法和类方法内存地址不一样了.

结论: 造成这种地址相同错觉的罪魁祸首就是Python的内存管理机制... 实际上他们是不同的对象, 只是编译器对内存做了重用. 以上就是我对这个问题的理解 - - 有错误希望大佬指出!_(:зゝ∠)_


