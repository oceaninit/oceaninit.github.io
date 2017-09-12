---
layout: post
title:  "Python slots"
date:   2017-09-12
categories: python
---

## Python `__slots__`


```python
class A(object):
    __slots__ = ('v')
```

如果在类中定义了`__slots__`属性，那么类的实例将不会拥有`__dict__`属性，也不能再添加实例属性。`__slots__`的作用就是限制实例能添加的属性。

`__slots__`作用：

  * 速度更快？
  * 节省内存

##实现

**`__slots__`中的变量是类属性，类型为数据描述符。**`__slots__`中的变量虽然是类属性，但是不同实例之间互不影响, 这是由描述符的实现决定的，倘若描述符实现时未考虑这点，则是所有实例共用的。


##Tips

* `__slots__`仅对当前类起作用，对子类不起作用，除非在子类中也定义`__slots__`，子类允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。
* 实例在未给`__slots__`中的变量赋值前，不能使用变量。
* 如果`__slots__`中的变量为类变量，该变量对于实例来说是只读的。如果想修改的话，可以通过类来修改。
```python
class B(object):
    __slots__ = ('v')

    v = 1

>>> b =  B()
>>> b.v
1
>>> b.v = 2

Traceback (most recent call last):
  File "<pyshell#8>", line 1, in <module>
    b.v = 2
AttributeError: 'B' object attribute 'v' is read-only    
```
* <http://tool.oschina.net/uploads/apidocs/python2.7.3/reference/datamodel.html>

* 如果需要给实例动态增加属性，将 `__dict__`添加到 `__slots__`。
* 定义`__slots__`的实例不支持弱引用，如需支持，需要将`__weakref__`添加到 `__slots__`。

```python
class C(object):
    __slots__ = ('v', '__weakref__')

    v = 1  

>>> b = B()
>>> import weakref
>>> weakref.ref(b)

Traceback (most recent call last):
  File "<pyshell#3>", line 1, in <module>
    weakref.ref(b)
TypeError: cannot create weak reference to 'B' object
>>> c = C()
>>> weakref.ref(c)
<weakref at 0000000002FD1E08; to 'C' at 0000000002FDAEB8>    
```

`__weakref__` is the head of the internal linked list of all the weak references to the object.

> The garbage collector completely ignores weakrefs when collecting reachable objects. A weakref simply doesn't count as a reference from the viewpoint of the garbage collector. Weakrefs are only considered when cleaning up. The GC traverses the list of weakrefs, invalidates them and calls any callback on the weakref. 

```
>>> r = weakref.ref(c)
>>> r
<weakref at 0000000002ED1E58; to 'C' at 0000000002EDAEB8>
>>> c.__weakref__
<weakref at 0000000002ED1E58; to 'C' at 0000000002EDAEB8>

>>> rr = weakref.ref(c)
>>> rr
<weakref at 0000000002ED1E58; to 'C' at 0000000002EDAEB8>  ## **调用weakref.ref得到的 r 和 rr 竟然是同一个对象！**
>>> c.__weakref__
<weakref at 0000000002ED1E58; to 'C' at 0000000002EDAEB8>

>>> weakref.getweakrefcount(c)
1
>>> weakref.getweakrefs(c)
[<weakref at 0000000002ED1E58; to 'C' at 0000000002EDAEB8>]

>>> import sys
>>> sys.getrefcount(c)
2

当使用某个引用作为参数，传递给getrefcount()时，参数实际上创建了一个临时的引用。因此，getrefcount()所得到的结果，会比期望的多1。
```