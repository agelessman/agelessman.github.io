---
layout:     post
title:      python中super()的一些用法
subtitle:   
date:       2018-07-24
author:     machao
header-img: 
catalog: true
tags:
- python
---

在看python高级编程这本书的时候，在讲到super的时候，产生了一些疑惑，super在python中的用法跟其他的语言有一些不一样的地方，在网上找了一些资料，发现基本上很少有文章能把我的疑惑讲明白，其中[这篇文章](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/)最有价值的地方是它讲解了我们平时对super的正确使用方法。

首先看一段程序：

```python
class A:
    def __init__(self):
        print("A", end=" ")
        super().__init__()


class E:
    def __init__(self):
        print("E", end=" ")
        super().__init__()


class B(E):
    def __init__(self):
        print("B", end=" ")
        super().__init__()


class D:
    def __init__(self):
        print("D", end=" ")
        super().__init__()


class C(A, B, D):
    def __init__(self):
        print("C", end=" ")
        A.__init__(self)
        D.__init__(self)
        B.__init__(self)


print("MRO:", [x.__name__ for x in C.__mro__])
print(C())
```

这段程序的打印结果是：

```
MRO: ['C', 'A', 'B', 'E', 'D', 'object']
C A B E D D B E D <__main__.C object at 0x1040340b8>
```

这段程序可以简单演示super()的用法，比较简单的概念是__mro__，它告诉我们某个类的继承链信息，生成这个链的原则有两条：

- 在继承关系中，所有的子类都在父类前边
- 如果出现冲突，则按照其__base__排序


要想理解上边程序的打印结果，只需要知道super()其实它相当于super(cls, self)，知道了这一点答案就很清晰了

当执行`A.__init__(self)`这行代码的时候，self指向了c实例对象，然后打印出A，当调用`super().__init__()`这行代码的时候，实际相当于`super(A, self).__init__()`的调用， `super(A, self)`的会在self.__mro__中查找排在A后边的类，这样就实现了继承链的调用了。

其实到了这里，我们基本上能猜到super()的实现原理了，虽然看上去它像没传入参数一样，其实不然，如果我们改变参数，找到的父类是不同的。

我们把上边代码中的A稍微改变一下：

```python
class A:
    def __init__(self):
        print("A", end=" ")
        super(D, self).__init__()

```

那么在这里`super(D, self)`获取的就是E了，因此打印结果是：

```python
MRO: ['C', 'A', 'B', 'E', 'D', 'object']
C A D B E D <__main__.C object at 0x1040340b8>
```

super()具有运行时特性，也就是说它的结果是动态计算出来的，利用这一点可以写出扩展性比较好的程序，同时，从编程语义也更符合生活习惯。目前能够总结的就是这样。

			
