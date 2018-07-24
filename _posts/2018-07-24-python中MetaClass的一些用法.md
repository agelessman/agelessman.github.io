---
layout:     post
title:      python中MetaClass的一些用法
subtitle:   
date:       2018-07-24
author:     machao
header-img: 
catalog: false
tags:
- python
---



元类在很多编程语言中都有这样的概念，我们都知道，类可以创建对象，类本身也是对象，既然是对象，那么它肯定也是被创造出来的，元类就专门用来创造类对象，于是，这就给我们提供了一种操纵或者监听类的能力。

平时我们创建一个类，使用的是这种方式：

```python
class MyClass(object):
    def method(self):
        return 1


instance1 = MyClass()
print(instance1.method())
```

如果把类也看成一个对象，利用元类，我们这样创建一个类：

```python
def method(self):
    return 1


klass = type('MyClass', (object,), {'method': method})
instance1 = klass()
print(instance1.method())
```

如果从写法上看，没什么太大的区别，MetaClass真正有用的地方是，我们可以自定义元类，接下来我们看一段代码：

```python
class RevealingMeta(type):
    def __new__(mcs, name, bases, namespace, **kwargs):
        print(mcs, "__new__ called")
        return super().__new__(mcs, name, bases, namespace)

    @classmethod
    def __prepare__(metacls, name, bases, **kwargs):
        print(metacls, "__prepare__ called")
        return super().__prepare__(name, bases, **kwargs)

    def __init__(cls, name, bases, namespace, **kwargs):
        print(cls, "__init__ called")
        super().__init__(name, bases, namespace)

    def __call__(cls, *args, **kwargs):
        print(cls, "__call__ called")
        return super().__call__(*args, **kwargs)

```

上边的代码可以算是固定写法了，这其中比较重要的概念是这4个方法的调用时机：

- `__new__ ` 当某个类被创建的时候会调用
- `__prepare__ ` 在创建类的时候，可以传入额外的字典`class Klass(metaclass=Metaclass, extra="value"):`,这个方法就是用来创建接收dict的，所以这个方法会在`__new__ `前边调用
- `__init__ ` 这个方法算是对`__new__ `的一些补充
- `__call__ ` 这个方法会在类被创建的时候调用

我们就利用上边代码创建出来的元类，创建一个类：

```python
class RevealingMeta(type):
    def __new__(mcs, name, bases, namespace, **kwargs):
        print(mcs, "__new__ called")
        return super().__new__(mcs, name, bases, namespace)

    @classmethod
    def __prepare__(metacls, name, bases, **kwargs):
        print(metacls, "__prepare__ called")
        return super().__prepare__(name, bases, **kwargs)

    def __init__(cls, name, bases, namespace, **kwargs):
        print(cls, "__init__ called")
        super().__init__(name, bases, namespace)

    def __call__(cls, *args, **kwargs):
        print(cls, "__call__ called")
        return super().__call__(*args, **kwargs)


class RevealingClass(metaclass=RevealingMeta):
    def __new__(cls, *args, **kwargs):
        print(cls, "__new__ called")
        return super().__new__(cls)

    def __init__(self):
        print(self, "__init__ called")
        super().__init__()
```

如果这时候直接执行代码，会打印出：

```python
<class '__main__.RevealingMeta'> __prepare__ called
<class '__main__.RevealingMeta'> __new__ called
<class '__main__.RevealingClass'> __init__ called
<class '__main__.RevealingClass'> __call__ called
```

这说明，只要创建了类就会调用上边的方法，这些方法的调用跟实例的创建没有关系，接下来执行：

```python
instance12 = RevealingClass()
```

在上边打印的后边，会打印出：

```python
<class '__main__.RevealingClass'> __new__ called
<__main__.RevealingClass object at 0x104032048> __init__ called
```

这就是元类的基本用法了，**它一般会用在很多的framework中，但也容易出错，我们可以为某个类随意指定元类，如果该元类没有实现这些方法，就有可能会出现崩溃。**
