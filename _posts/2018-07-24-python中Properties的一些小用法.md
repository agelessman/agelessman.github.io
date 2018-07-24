---
layout:     post
title:      python中Properties的一些小用法
subtitle:   
date:       2018-07-24
author:     machao
header-img: 
catalog: false
tags:
- 操作系统
---


property最大的用处就是可以为一个属性制定getter，setter，delete和doc，他的函数原型为：

```python
    def __init__(self, fget=None, fset=None, fdel=None, doc=None): # known special case of property.__init__
        """
        property(fget=None, fset=None, fdel=None, doc=None) -> property attribute
        
        fget is a function to be used for getting an attribute value, and likewise
        fset is a function for setting, and fdel a function for del'ing, an
        attribute.  Typical use is to define a managed attribute x:
        
        class C(object):
            def getx(self): return self._x
            def setx(self, value): self._x = value
            def delx(self): del self._x
            x = property(getx, setx, delx, "I'm the 'x' property.")
        
        Decorators make defining new properties or modifying existing ones easy:
        
        class C(object):
            @property
            def x(self):
                "I am the 'x' property."
                return self._x
            @x.setter
            def x(self, value):
                self._x = value
            @x.deleter
            def x(self):
                del self._x
        
        # (copied from class doc)
        """
        pass
```

从上边的代码中可以看出来，它一共接受4个参数，我们再继续看一段代码：

```python
class Rectangle(object):
    def __init__(self, x1, y1, x2, y2):
        self.x1, self.y1 = x1, y1
        self.x2, self.y2 = x2, y2

    def _width_get(self):
        return self.x2 - self.x1

    def _width_set(self, value):
        self.x2 = self.x1 + value

    def _height_get(self):
        return self.y2 - self.y1

    def _height_set(self, value):
        self.y2 = self.y1 + value

    width = property(_width_get, _width_set, doc="rectangle width measured from left")
    height = property(_height_get, _height_set, doc="rectangle height measured from top")

    def __repr__(self):
        return "{}({}, {}, {}, {})".format(self.__class__.__name__,
                                           self.x1,
                                           self.y1,
                                           self.x2,
                                           self.y2)


rectangle = Rectangle(10, 10, 30, 15)
print(rectangle.width, rectangle.height)
rectangle.width = 50
print(rectangle)
rectangle.height = 50
print(rectangle)
print(help(rectangle))
```

通过property，我们有能力创造出一个属性来，然后为这个属性指定一些方法，**在这里用setter，getter的好处就是可以监听属性的赋值和获取行为**，表面上看上去上边的代码没有问题，但是当出现继承关系的时候，就出问题了。

```python
class MetricRectangle(Rectangle):
    def _width_get(self):
        return "{} metric".format(self.x2 - self.x1)


mr = MetricRectangle(10, 10, 100, 100)
print(mr.width)
```

即使我们在子类中重写了getter方法，结果却是无效的，这说明property只对当前的类生效，于是不得不把代码改成下边这样：

```python
class MetricRectangle(Rectangle):
    def _width_get(self):
        return "{} metric".format(self.x2 - self.x1)

    width = property(_width_get, Rectangle.width.fset)


mr = MetricRectangle(10, 10, 100, 100)
print(mr.width)

```

**因此，在平时的编程中，如果需要重写属性的话，应该重写该类中所有的property，否则程序很很难以理解，试想一下，setter在子类，getter在父类，多么恐怖**，

另一种比较好的方案是使用装饰器，可读性也比较好

```python 
class Rectangle(object):
    def __init__(self, x1, y1, x2, y2):
        self.x1, self.y1 = x1, y1
        self.x2, self.y2 = x2, y2

    @property
    def width(self):
        """rectangle width measured from left"""
        return self.x2 - self.x1

    @width.setter
    def width(self, value):
        self.x2 = self.x1 + value

    @property
    def height(self):
        return self.y2 - self.y1

    @height.setter
    def height(self, value):
        self.y2 = self.y1 + value

    def __repr__(self):
        return "{}({}, {}, {}, {})".format(self.__class__.__name__,
                                           self.x1,
                                           self.y1,
                                           self.x2,
                                           self.y2)


rectangle = Rectangle(10, 10, 30, 15)
print(rectangle.width, rectangle.height)
rectangle.width = 50
print(rectangle)
rectangle.height = 50
print(rectangle)
print(help(rectangle))
```
