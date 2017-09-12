---
layout: post
title:  "Python descriptors"
date:   2017-09-12
categories: python
---

## Python descriptors

* 描述符就是一个绑定方法`__get__`(), `__set__`(), `__delete__`()的对象。
```
descriptor.__get__(self, obj, type=None) --> value
descriptor.__set__(self, obj, value) --> None
descriptor.__delete__(self, obj) --> None
```
上述方法就是描述符方法，如果一个对象定义了描述符方法中的任何一个，那么这个对象就会成为描述符。

* 当描述符作为类属性时，才会自动调用描述符方法，描述符作为实例属性时，不会自动调用描述符方法。
```
class Foo(object):
    y = descriptor(0)
    def __init__(self):
        self.x = descriptor(1)       #实例属性的描述符是不会自动调用对应的描述符方法的
```

* 描述符的自动调用机制基于`__getattribute__()`, `__getattribute__`()确保了descriptor的机制，所以，如果重写了`__getattribute__`, 就会消除descriptor机制。
* 描述符是所有实例共享的, 如果实例修改了，可能会影响其它实例，这就涉及到描述符的实现：

    * 数据字典法。
    在descriptor中使用数据字典，由`__get__`和`__set__`的第一个参数来确定是哪个实例，使用实例作为字典的key，为每一个实例单独保存一份数据，缺陷就是不可哈希对象不能作为键值。


    * 标签法。
    给实例增加一个与描述符同名的实例属性，利用该实例属性来保存该实例描述符的值，
    描述符相当于一个中间操作，描述符的`__get__`()返回实例属性，`__set__`也是对实例属性操作。原理: 数据描述符的访问优先级比实例属性高.

<http://blog.csdn.net/lis_12/article/details/53453665>
 
```python
from weakref import WeakKeyDictionary
class descriptor(object):
    def __init__(self, default):
        self.default = default
        self.data = WeakKeyDictionary()

    def __get__(self, instance, owner):# instance = x,owner = type(x)
        # we get here when someone calls x.d, and d is a descriptor instance
        return self.data.get(instance, self.default)

    def __set__(self, instance, value):
        # we get here when someone calls x.d = val, and d is a descriptor instance
        self.data[instance] = value

class Foo(object):
    bar = descriptor(5)

f = Foo()
g = Foo()
print "f.bar is %s g.bar is %s" % (f.bar, g.bar)   #f.bar is 5  g.bar is 5
print "Setting f.bar to 10"
f.bar = 10
print "f.bar is %s\ng.bar is %s" % (f.bar, g.bar)  ##f.bar is 10  g.bar is 5
```


```

class descriptor(object):
    def __init__(self, label):#label为给实例增加的实例属性名
        self.label = label
    def __get__(self, instance, owner):
        #dict.get(k[,d]) -> D[k] if k in D, else d.  d defaults to None.
        return instance.__dict__.get(self.label)  #获取与描述符同名的实例属性的值 

    def __set__(self, instance, value):
        #注意这里,要这么写,不能写instance.x = val这种形式,这样会形成自身的循环调用
        instance.__dict__[self.label] = value     #修改与描述符同名的实例属性的值  

class Foo(list):
    x = descriptor('x') #注意这个初始化值为要给实例增加的实例属性名，要和描述符对象同名。
    y = descriptor('y')

f1 = Foo()
f2 = Foo()
print f1.__dict__  #{}
print f1.x,f2.x,f1.y,f2.y#None None None None,此时尚未增加实例属性,需要调用__set__方法建立一个与描述符同名的实例属性
#print Foo.__dict__
f1.x = 1
f1.y = 2
f2.x = 3
f2.y = 4           
print f1.__dict__  #{'y': 2, 'x': 1}  #增加了的实例属性
print f1.x,f1.y,f2.x,f2.y    #1 2 3 4 
```

因为只有调用了__set__函数才会建立一个与描述符同名的实例属性，所以可以在__init__()函数中对描述符赋值。
```

class Foo(list):
    x = descriptor('x')
    y = descriptor('y')
    def __init__(self):
        self.x = 1  #调用的是描述符的__set__方法,与描述符同名的实例属性增加完毕....
        self.y = 2
f = Foo()
print f.x,f.y 
```