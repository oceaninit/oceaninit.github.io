---
layout: post
title:  "Python assignment"
date:   2017-08-03 22:49:02 +0700
categories: python
---

## python 遍历删除list中的元素

```python
a = [1, 2, 3, 4, 5]
for i in a:
	if i % 2 == 0:
		a.remove(i)
	print a, i

a = [1, 2, 3, 4, 5]
for i, d in enumerate(a):
	if d % 2 == 0:
		del a[i]
	print a, i, d


a = [1, 2, 3, 4, 5]
for i in range(len(a)):
	if a[i] == 4:
		del a[i]
	print a, i


###output 
# [1, 2, 3, 4, 5] 1
# [1, 3, 4, 5] 2
# [1, 3, 5] 4

# [1, 2, 3, 4, 5] 0 1
# [1, 3, 4, 5] 1 2
# [1, 3, 5] 2 4

# [1, 2, 3, 4, 5] 0
# [1, 2, 3, 4, 5] 1
# [1, 2, 3, 4, 5] 2
# [1, 2, 3, 5] 3
# Traceback (most recent call last):
#   File "G:\Python\CodeSlice\python\list_remove.py", line 16, in <module>
#     if a[i] == 4:
# IndexError: list index out of range
```

for 和 enumerate本质是使用了迭代器，`__iter__`可以拿到一个迭代器，可以手动调用next，

```
>>> a= [1, 2, 3, 4, 5]
>>> a.__iter__()
<listiterator object at 0x0000000002ED1780>
>>> ia = a.__iter__()
>>> ia.next()
1
>>> ia.next()
2
```

通过iter获得一个迭代器对象，要求a实现`__iter__`, 并返回一个迭代器对象
```
>>> it = iter(a)
>>> it
<listiterator object at 0x0000000002ED1780>
```

enumerate(iterator, start)等效为：(生成器的形式)
```
def enumerate(iterator, start=0):
    n = start
    for elem in iterator:
        yield n, elem
        n += 1
```