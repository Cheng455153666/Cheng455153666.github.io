---
title: Python中的新式&经典类
date: 2018.05.25 00:47:43
categories: Python
tags: [Python]
comment: false
---

## 新式/经典类的区别

从Python2.2开始,Python 引入了 new style class(新式类)新式类, 这里简单的说一下相对于经典类, 新式类的区别.

下面直接上代码:

```Python
# -*- coding:utf-8 -*-


class old_class:  # 经典类
    pass


class new_class:  # 新式类
    pass


old = old_class()
new = new_class()

print '经典类'
print old
print type(old)
print old.__class__
print dir(old)

print "新式类"
print new
print new.__class__
print dir(new)
print type(new)
```

在python2.7环境下查看一下输出

```shell
经典类
<__main__.old_class instance at 0x10e14ccb0>
<type 'instance'>
__main__.old_class
['__doc__', '__module__']
新式类
<__main__.new_class object at 0x10e12d410>
<class '__main__.new_class'>
['__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
<class '__main__.new_class'>
```

区别已经很明显了, 除去类型不同之外,新式类多出了许多内置方法. 这里就不赘述用法了.

还有一个重要的区别在于, 新式类和经典类对方法查找方式有区别. 我们称之为广度优先和深度优先. 但很多人会误会这句话的意思.

我们来看一个例子:

![rOkm8L](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620610262114/rOkm8L.jpg)

大家可能会理解为这种查找方式, 但是实际上, 不是的, 如果方法没找到的情况下, 且只有一个条继承的路线, 那么就会沿着这条路线继续向上找. 说的有点晕乎, 我们看例子:

![V2wwfu](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620610272106/V2wwfu.jpg)

这里我们可以这么理解, E 继承自B, C, D 而B, C, D又同时继承自A, 所以查找的思路是这样的.

* E 首先找到B, 发现B 与C, D共享A, 就不再向上找, 横向查找.
* E 于是找到了C, 发现 C 与 D共享A, 于是也不再向上找, 又开始横向查找
* E 再找到了D, 此时只有D这么最后一根线连着A了, 便开始沿着D的继承关系, 找到了A
如果到这里, 你已经明白了, 可以尝试着看看这题:

![MXB7oS](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620610288228/MXB7oS.jpg)


下面是这道题的答案, 在分析一下

1. A先找到B, B继承自E且只有B继承自E, 所以找E , 同理发现E继承且只有E继承自H , 于是找H , 再找到J
2. 再开始找C ,发现C继承且仅有C继承自F, 于是开始找F, 开始找F, 发现F与I共同继承自K, 于是放弃向上找
3. 开始找D, D上与之前类似, 都是继承且仅仅只有一个继承关系, 一直到I, 发现除去刚刚找过的F之外, 已经没有其他类继承自K了, 于是I就向上走, 找到K

![Qn5BFi](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620610317795/Qn5BFi.jpg)

再来试试

![YiO50O](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620610326113/YiO50O.jpg)

又是答案..

![dYslEx](http://ckopenbucket.oss-cn-beijing.aliyuncs.com//1620610333874/dYslEx.jpg)

如果到这里, 你都对了, 那么新式类的方法查找路径你就理解了.

现在再来说说经典类. 就是一句话, 只要有爹就一路向上走.

我们按照经典类的方法查找路径去写写刚刚这道题.

这里给出答案 A B C E F G D E F G.

总结一下方法查找的区别:

经典类: 只要有父类, 就会沿着一直找, 即使已经找过了~

新式类: 在类继承的多个类拥有共同父类的情况下, 会优先横向查找, 直到剩下最后一个继承自这个共同父类的对象, 再向上查找.