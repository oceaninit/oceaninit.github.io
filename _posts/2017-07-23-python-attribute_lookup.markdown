---
layout: post
title:  "Python attribute lookup"
date:   2017-07-23
categories: python
---


## 作用域

  python变量访问顺序按照LEGB规则，即：

  * L->Local. 局部变量，如定义在def或lambda中的变量。
  * E-> Enclosing function locals. 嵌套的父级函数作用域，如闭包中的变量。
  * G->Global (module). 全局变量。
  * B->Built-in. 内置变量，如len等。


## *对象*属性查找

* ```__getattribute__```. 访问对象的任何属性时，都会隐式调用，```o.__dict__```内部会调用```o.__getattribute__('__dict__') ```。如果在重载```__getattribute__```中又调用```__dict__```的话, 会导致无限递归，解决就是用```object.__getattribute__(self, key)```。以下摘自[Descriptor HowTo Guide](https://docs.python.org/2/howto/descriptor.html?highlight=__getattribute__)。

```python
def __getattribute__(self, key):
    "Emulate type_getattro() in Objects/typeobject.c"
    v = object.__getattribute__(self, key)
    if hasattr(v, '__get__'):
        return v.__get__(None, self)
    return v
```
>The important points to remember are:

- ```descriptors are invoked by the __getattribute__() method```
- ```overriding __getattribute__() prevents automatic descriptor calls```
- ```__getattribute__() is only available with new style classes and objects```
- ```object.__getattribute__() and type.__getattribute__() make different calls to __get__().```
- ```data descriptors always override instance dictionaries.```
- ```non-data descriptors may be overridden by instance dictionaries.```

> If an object defines both ```__get__```() and ```__set__```(), it is considered a data descriptor. Descriptors that only define ```__get__```() are called non-data descriptors (they are typically used for methods but other uses are possible).

> Data and non-data descriptors differ in how overrides are calculated with respect to entries in an instance’s dictionary. If an instance’s dictionary has an entry with the same name as a data descriptor, the data descriptor takes precedence. If an instance’s dictionary has an entry with the same name as a non-data descriptor, the dictionary entry takes precedence.

> To make a read-only data descriptor, define both ```__get__```() and ```__set__```() with the ```__set__```() raising an AttributeError when called. Defining the ```__set__```() method with an exception raising placeholder is enough to make it a data descriptor.


* `__getattr__`, `__getattribute__`找不到时, 会调用`__getattr__`。未定义，抛出AttributeError异常。

* `__get__` 定义描述符Descriptor。

例如，obj = Cls(), 那么obj.attr查找顺序如下：

1. 如果attr出现在Cls或其基类(先Cls后基类)的```__dict__```中，且attr是data descriptor，那么调用其__get__方法, 否则
2. 如果attr出现在```obj.__dict__```中， 那么直接返回 ```obj.__dict__['attr']```，否则
3. 如果attr出现在Cls或其基类的```__dict__```中

	3.1 如果attr是non-data descriptor，那么调用其```__get__```方法, 否则
	
	3.2 返回 ```__dict__['attr']```

4. 如果Cls有```__getattr__```方法，调用```__getattr__```方法，否则
5. 抛出AttributeError异常

 > 先检查对象(类和基类)的数据描述符(data descriptor)，再检查实例字典```__dict__```，再检查类和基类的非数据描述符(non-data descriptor)，最后是类和基类的字典。**归结起来就是和`___dict___`以及数据描述符打交道。**如果指定了`__slots__`的话不会创建`__dict__`。概括起来：***类属性 > 数据描述符 > 实例属性 > 非数据描述符 -> `__getter__`() ***。

 > 在类实例中查找属性的时候，首先在实例自己的作用域中查找，如果没有找到，则再在类定义的作用域中查找。在对类实例属性进行赋值的时候，实际上会在类实例定义的作用域中添加一个属性（如果还不存在的话），并不会影响到相应类中定义的同名属性。


 ***如果重定义了`__getattribute__`, 还是上面这样吗？？？ ***, 下例，所有的属性都可调用，但结果都为None。
```python

class A(object):
	a = 2
	def __getattribute__(self, key):
		pass
o = A()
print o.a, o.ooxx
```

## 属性赋值时的查找策略

对于obj.attr = value, 基本同上

1. 在`obj.__class__.__dict__`中查找attr，如果存在并且是data descriptor，调用attr的`__set__`方法，否则
2. 继续到`obj.__class__`的父类和基类中查找，找到 data descriptor则调用其`__set__`方法，否则
3. 直接在`obj.__dict__`中加入`obj.__dict__['attr'] = value`

举个例子，对于data描述符，其优先级高于实例属性，赋值操作被`__set__`截获，实例的`__dict__`仍然是空的。

```python
class Descriptor(object):

	def __init__(self, value = None):
		self.value = value

	def __get__(self, instance, owner):
		return self.value

	def __set__(self, instance, value):
		self.value = value

class A(object):
	d = Descriptor()

>>> a = A()
>>> a.__dict__
{}
>>> a.__class__.__dict__
dict_proxy({'__dict__': <attribute '__dict__' of 'A' objects>, '__module__': '__main__', '__weakref__': <attribute '__weakref__' of 'A' objects>, 'd': <__main__.Descriptor object at 0x0000000003016898>, '__doc__': None})
>>> a.d
Descriptor.__get__ <__main__.A object at 0x00000000030167B8> <class '__main__.A'>
>>> a.d = 3
>>> a.d
3
>>> a.__dict__
{}
>>> a.__class__.__dict__
dict_proxy({'__dict__': <attribute '__dict__' of 'A' objects>, '__module__': '__main__', '__weakref__': <attribute '__weakref__' of 'A' objects>, 'd': <__main__.Descriptor object at 0x0000000003016898>, '__doc__': None})
>>> 

```
对于non-data描述符，
```python
class NonDescriptor(object):

	def __init__(self, value = None):
		self.value = value

	def __get__(self, instance, owner):
		print 'Descriptor.__get__', instance, owner


class B(object):
	d = NonDescriptor()

>>> b = B()
>>> b.__dict__
{}
>>> b.__class__.__dict__
dict_proxy({'__dict__': <attribute '__dict__' of 'B' objects>, '__module__': '__main__', '__weakref__': <attribute '__weakref__' of 'B' objects>, 'd': <__main__.NonDescriptor object at 0x00000000030168D0>, '__doc__': None})
>>> b.d = 3
>>> b.d
3
>>> b.__dict__
{'d': 3}
```
特殊的，直接通过类调用描述符，不会进入`__set__`，`B.__dict__`中的属性直接更新，也即意味着类属性的优先级高于描述符。
```python
>>> B.d = 2
>>> B.__dict__
dict_proxy({'__dict__': <attribute '__dict__' of 'B' objects>, '__module__': '__main__', '__weakref__': <attribute '__weakref__' of 'B' objects>, 'd': 2, '__doc__': None})
>>> B.d
2
```


## 继承属性查找顺序(MRO)

MRO的全称是Method Resolution Order，即方法解析顺序，用来定义类继承链中如何对类属性和方法进行查找，保证属性查找不会  
出现冲突。现在实际采用的是C3算法.可以通过类的`__mro__`属性和`mro`方法获得类的属性查找顺序。

```python
def super(cls, inst):
    mro = inst.__class__.mro()
    return mro[mro.index(cls)+1]
```

## Properties

`property(fget=None, fset=None, fdel=None, doc=None) -> property attribute`

`__get__`定义的描述符和property的区别是什么？
propery 定义了`__get__`和`__set__`, 属于data descriptor, 详见[Descriptor HowTo Guide](https://docs.python.org/2/howto/descriptor.html?highlight=__getattribute__)。

> Calling property() is a succinct way of building a data descriptor that triggers function calls upon access to an attribute.

## Functions and Methods

> To support method calls, functions include the `__get__`() method for binding methods during attribute access. This means that all functions are non-data descriptors which return bound or unbound methods depending whether they are invoked from an object or a class. In pure python, it works like this:

```python
class Function(object):

    def __get__(self, obj, objtype=None):
        "Simulate func_descr_get() in Objects/funcobject.c"
        return types.MethodType(self, obj, objtype)


>>> class D(object):
...     def f(self, x):
...         return x
...
>>> d = D()
>>> D.__dict__['f']  # Stored internally as a function
<function f at 0x00C45070>
>>> D.f              # Get from a class becomes an unbound method
<unbound method D.f>
>>> d.f              # Get from an instance becomes a bound method
<bound method D.f of <__main__.D object at 0x00B18C90>>
```


<!-- 更多可以参考这个 https://segmentfault.com/a/1190000008994270 -->




