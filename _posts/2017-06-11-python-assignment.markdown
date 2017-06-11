---
layout: post
title:  "Python assignment"
date:   2017-06-11 18:28:02 +0700
categories: python
---

python链式赋值的两个函数：

```python
def assign1():
    s = [1, 2, 3, 4, 5, 6]
	i = 0
	i = s[i] = 3

# 输出：
# >>> s
# [1, 2, 3, 3, 5, 6]
```

```python
def assign2():
	s = [1, 2, 3, 4, 5, 6]
	i = 0
	s[i] = i = 3

# 输出：
# >>> s
# [3, 2, 3, 4, 5, 6]
```

使用dis窥探下内部实现

```
>>> dis.dis(assign1)
 17           0 LOAD_CONST               1 (1)
              3 LOAD_CONST               2 (2)
              6 LOAD_CONST               3 (3)
              9 LOAD_CONST               4 (4)
             12 LOAD_CONST               5 (5)
             15 LOAD_CONST               6 (6)
             18 BUILD_LIST               6
             21 STORE_FAST               0 (s)

 18          24 LOAD_CONST               7 (0)
             27 STORE_FAST               1 (i)

 19          30 LOAD_CONST               3 (3)
             33 DUP_TOP             
             34 STORE_FAST               1 (i)
             37 LOAD_FAST                0 (s)
             40 LOAD_FAST                1 (i)
             43 STORE_SUBSCR        
             44 LOAD_CONST               0 (None)
             47 RETURN_VALUE 
```

```
>>> dis.dis(assign2)
 26           0 LOAD_CONST               1 (1)
              3 LOAD_CONST               2 (2)
              6 LOAD_CONST               3 (3)
              9 LOAD_CONST               4 (4)
             12 LOAD_CONST               5 (5)
             15 LOAD_CONST               6 (6)
             18 BUILD_LIST               6
             21 STORE_FAST               0 (s)

 27          24 LOAD_CONST               7 (0)
             27 STORE_FAST               1 (i)

 28          30 LOAD_CONST               3 (3)
             33 DUP_TOP             
             34 LOAD_FAST                0 (s)
             37 LOAD_FAST                1 (i)
             40 STORE_SUBSCR        
             41 STORE_FAST               1 (i)
             44 LOAD_CONST               0 (None)
             47 RETURN_VALUE        
>>> 
```

a = 3
a, b = 1, a
如果按照正常的思维逻辑，先进行a = 1，在进行b = a，最后b应该等于1，但是这里b应该等于3，因为在连续赋值语句中等式右边其实都是局部变量，而不是真正的变量值本身，比如，上面例子中右边的a，在python解析的时候，只是把变量a的指向的变量3赋给b，而不是a=1之后a的结果，这一点刚开始学python的人可能容易误解，再举一个Leetcode里链表的例子理解就更深了。
