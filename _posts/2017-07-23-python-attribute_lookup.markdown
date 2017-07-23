---
layout: post
title:  "Python attribute lookup"
date:   2017-07-23
categories: python
---

# python 属性查找

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


* ```__getattr__ ```. ```__getattribute__```找不到时, 会调用```__getattr__```。未定义，抛出AttributeError异常。

* ```__get__```. 定义描述符Descriptor。

例如，obj = Cls(), 那么obj.attr查找顺序如下：

1. 如果attr出现在Cls或其基类(先Cls后基类)的```__dict__```中，且attr是data descriptor，那么调用其__get__方法, 否则
2. 如果attr出现在```obj.__dict__```中， 那么直接返回 ```obj.__dict__['attr']```，否则
3. 如果attr出现在Cls或其基类的```__dict__```中

    3.1 如果attr是non-data descriptor，那么调用其```__get__```方法, 否则

  	3.2 返回 ```__dict__['attr']```

 4. 如果Cls有```__getattr__```方法，调用```__getattr__```方法，否则
 5. 抛出AttributeError异常

 > 先检查对象(类和基类)的数据描述符(data descriptor)，再检查实例字典```__dict__```，再检查类和基类的非数据描述符(non-data descriptor)，最后是类和基类的字典。