---
layout: post
title:  "Python descriptors"
date:   2017-09-12
categories: python
---

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

    * 标签法。

    给实例增加一个与描述符同名的实例属性，利用该实例属性来保存该实例描述符的值，
    描述符相当于一个中间操作，描述符的`__get__`()返回实例属性，`__set__`也是对实例属性操作。原理: 数据描述符的访问优先级比实例属性高.

```python
# 这是一个错误的实现！！！

class Descriptor1(object):
  def __init__(self, label):
    self.label = label

  def __get__(self, instance, owner):
    return getattr(instance, self.label)

  def __set__(self, instance, value):
    setattr(instance, self.label, value)

class C(object):
  a = Descriptor1('a')

  def __init__(self, val):
    self.a = val

>>> c = C(1)

Traceback (most recent call last):
  File "<pyshell#9>", line 1, in <module>
    c = C(1)
  File "G:\Python\CodeSlice\python\descriptor.py", line 47, in __init__
    self.a = val
  File "G:\Python\CodeSlice\python\descriptor.py", line 41, in __set__
    setattr(instance, self.label, value)
  .
  .
  .

  File "G:\Python\CodeSlice\python\descriptor.py", line 41, in __set__
    setattr(instance, self.label, value)
  RuntimeError: maximum recursion depth exceeded while calling a Python object

```

---

```python
# 正确的实现

class Descriptor2(object):
  def __init__(self, label):#label为给实例增加的实例属性名
    self.label = label

  def __get__(self, instance, owner):
    return instance.__dict__.get(self.label)  #获取与描述符同名的实例属性的值 

  def __set__(self, instance, value):
    #不能写instance.x = val这种形式, 对自身的循环调用！！！
    instance.__dict__[self.label] = value     #修改与描述符同名的实例属性的值  

class D(object):
  a = Descriptor2('a') #注意这个初始化值为要给实例增加的实例属性名，要和描述符对象同名。

  def __init__(self, val): 
    '''
    只有调用了__set__函数才会建立一个与描述符同名的实例属性，所以可以在__init__()函数中对描述符赋值。
    无此函数访问x也没trace，打印为空。
    '''
    self.a = val  
```


* 如果同时定义了 `__slots__`和descriptor?

      定义了`__slots__`的实例是没有`__dict__`的，看看[Yahya Abou 'Imran][2]的思路

>So, here is an a generic solution that tries to acces to the `__dict__` variable first (which is the default anyway) and, if it fails, use getattr and setattr:

```python
class WorksWithDictAndSlotsDescriptor(object):
# class WorksWithDictAndSlotsDescriptor: 
# 旧式类，不可用, e.a = <__main__.WorksWithDictAndSlotsDescriptor instance at 0x0000000002E25408>

  def __init__(self, attr_name):
    self.attr_name = attr_name

  def __get__(self, instance, owner):
    try:
      return instance.__dict__[self.attr_name]
    except AttributeError:
      return getattr(instance, self.attr_name)

  def __set__(self, instance, value):
    try:
      instance.__dict__[self.attr_name] = value
    except AttributeError:
      setattr(instance, self.attr_name, value)
```

>Works only if the attr_name is not the same as the real instance variable's name, or you will have a RecursionError as pointed to in the accepted answer

```python

class E(object):
  __slots__ = ('a',)
  a = WorksWithDictAndSlotsDescriptor('a')

  def __init__(self, val): 
    self.a = val 

>>> e = E(1)

Traceback (most recent call last):
  File "<pyshell#0>", line 1, in <module>
    e = E(1)
  File "G:\Python\CodeSlice\python\descriptor.py", line 97, in __init__
    self.a = val
  File "G:\Python\CodeSlice\python\descriptor.py", line 88, in __set__
    setattr(instance, self.attr_name, value)
  File "G:\Python\CodeSlice\python\descriptor.py", line 88, in __set__

  .
  setattr(instance, self.attr_name, value)
  RuntimeError: maximum recursion depth exceeded while calling a Python object recursion depth exceeded 
  while calling a Python object
```

一种正确的用法：

```python

class E2(object):
  __slots__ = ('_a',)
  a = WorksWithDictAndSlotsDescriptor('_a')

  def __init__(self, val): 
    self.a = val 
```

同[Glenn Maynard][3]的答案



[1]: <http://blog.csdn.net/lis_12/article/details/53453665>
[2]: <https://stackoverflow.com/questions/4912499/using-python-descriptors-with-slots>
[3]: <https://stackoverflow.com/questions/4912499/using-python-descriptors-with-slots>