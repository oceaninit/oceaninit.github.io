---
layout: post
title:  "Python assignment"
date:   2017-06-11 18:28:02 +0700
categories: python
---

##链式赋值：

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
##swap操作：

```python
def assign3():
    a = 3
	a, b = 1, a

# >>> a, b
# (1, 3)
# >>> b is a
# False
# >>> id(b)
# 30440824L
# >>> id(a)
# 30440872L
```

```
>>> dis.dis(assign3)
 35           0 LOAD_CONST               1 (3)
              3 STORE_FAST               0 (a)

 36           6 LOAD_CONST               2 (1)
              9 LOAD_FAST                0 (a)
             12 ROT_TWO             
             13 STORE_FAST               0 (a)
             16 STORE_FAST               1 (b)
             19 LOAD_CONST               0 (None)
             22 RETURN_VALUE 
```    

dis结果显示，swap操作并不是从左至右，先把a赋值1，再把a的值赋给b，而是同时读取赋值符右边的数，然后赋值。在连续赋值语句中等式右边其实都是局部变量，而不是真正的变量值本身

```python
def assign4():
    a = 1
    b = 2
    a, b = b, a

# >>> a, b
# (2, 1)
# >>> a is b
# False
# >>> id(a)
# 31227280L
# >>> id(b)
# 31227304L
```

```
>>> dis.dis(assign4)
 43           0 LOAD_CONST               1 (1)
              3 STORE_FAST               0 (a)

 44           6 LOAD_CONST               2 (2)
              9 STORE_FAST               1 (b)

 45          12 LOAD_FAST                1 (b)
             15 LOAD_FAST                0 (a)
             18 ROT_TWO             
             19 STORE_FAST               0 (a)
             22 STORE_FAST               1 (b)
             25 LOAD_CONST               0 (None)
             28 RETURN_VALUE 
```