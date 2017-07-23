---
layout: post
title:  "Python singleton"
date:   2017-07-22
categories: python
---

## Python单例

* 模块，模块本身即是单例模式, 对象定义在模块内，直接import

* ```__new__```

```python

class Singleton(object):
	_instance = None

	def __new__(cls, *args, **kwargs): #__new__()必须要有返回值, 参数与__init__对应
		if not cls._instance:
			cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
		return cls._instance

class Singleton(object):
	def __new__(cls, *args, **kwargs): #__new__()必须要有返回值, 参数与__init__对应
		if not hasattr(cls, '_instance'):
			cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
		return cls._instance		

```		

* metaclass


	```__call__```使**类的对象**变成可调用对象，而不是改变类的实例化行为，然鹅元类可以，因类本身就是元类的实例，会改变类的实例化行为。 type的```__call__```实际上调用了type的```__new__```和```__init__```。调用```__call__```的时候，类已经被创建出来，```__call__```作用在类创建实例的过程。

```python

class Singleton1(type):

	# def __init__(self, name, bases, attrs):
	def __init__(self, *args, **kwargs):
		super(Singleton1, self).__init__(*args, **kwargs)
		self._instance = None

	def __call__(self, *args, **kwargs):
		if not self._instance:
			self._instance = super(Singleton1, self).__call__(*args, **kwargs)
		return self._instance

class MySingleton1(object):
	__metaclass__ = Singleton1

# class MySingleton1(metaclass = Singleton1)
# 	pass
```

```python
class Singleton2(type):
	_instances = {}

	def __call__(self, *args, **kwargs):
		if self not in self._instances:
			self._instances[self] = super(Singleton2, self).__call__(*args, **kwargs)
		return self._instances[self]

class MySingleton2(object):
	__metaclass__ = Singleton2
	

```

* 装饰器

```python

from functools import wraps

def singleton(cls):
	_instances = {} #{}装饰多个哦, 不想覆盖的话

	@wraps(cls)
	def wrapper(*args, **kwargs):
		if cls not in _instances:
			_instances[cls] = cls(*args, **kwargs)
		return _instances[cls]

	return wrapper

@singleton
class Singleton3(object):
	pass

```
	