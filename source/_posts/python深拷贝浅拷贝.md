---
title: Python deep copy and shallow copy
date: 2023-02-18 17:46:25
top: true
tags:
    - python
---
When it comes to object copying in Python, we need to understand the difference between is and ==:

```
>>> a = [1,2,3,4]
>>> b = [1,2,3,4]
>>> a == b
True
>>> a is b
False

```

From the example above, we can see that the values in a and b are equal, but the result of is is different. This is because Python compares whether the values of a and b are equal, while is compares whether the object identities are equal. Therefore, in Python, we usually use == to compare the values of objects and use is to determine whether the values bound to objects are None. It should be noted that some beginners often make the mistake of using None to compare whether a string, list, or dictionary is empty, which is incorrect because these variables are not equal to None in terms of object identity or value.

In Python, we usually use shallow copy to copy objects. The copy module provides us with the copy (shallow copy) and deepcopy (deep copy) functions.

Shallow copy refers to copying a reference to the copied object, and the copied object points to the value of the object being copied. In other words, a reference is added to the original value.

```
>>> a = 1
>>> b = a
>>> id(a)
4297636352
>>> id(b)
4297636352

```

Deep copy refers to copying the value of the copied object and creating a new object whose value and object identity are equal to the copied object.

```
>>> import copy
>>> a = [1,2,3]
>>> b = copy.deepcopy(a)
>>> b
[1,2,3]
>>> a
[1,2,3]
>>> a == b
True
>>> a is b
True
>>> id(a)
4297636352
>>> id(b)
4297636352
>>> a.append(0)
>>> a
[1,2,3,0]
>>> b
[1,2,3]
>>> b.append(9)
>>> b
[1, 2, 3, 9]
>>> a
[1, 2, 3, 0]

```

Based on this phenomenon, we should pay special attention to the use of mutable parameters as default parameters in functions. If we are not careful, we may encounter the following situation:

```
>>> def a(x = []):
...     x.append(0)
...     print(x)
>>> a()
[0]
>>> a()
[0, 0]
>>> a()
[0, 0, 0]

```

To avoid this situation, we should avoid using mutable objects as default function parameters:

```
>>> def a(x = None):
...     if x is None:
...         x = []
...     else:
...         x.append(0)
...     print(x)
...
>>> a()
[]
>>> a()
[]

```

At the same time, when creating a class and passing parameters during initialization, shallow copy is also used to pass parameters, which can lead to the following situation:

```
>>> class A:
...     def __init__(self, name):
...         self.name = name
...     def printf(self):
...         print(self.name)
...
>>> x = [1,2,3,4]
>>> a = A(x)
>>> a.printf()
[1, 2, 3, 4]
>>> x.append(5)
>>> a.printf()
[1, 2, 3, 4, 5]

```

When the passed-in parameter changes, the value of the variable in the class will also change, which is difficult to detect. The biggest difference between deep copy and shallow copy is that deep copy will copy the parent object and its sub-objects, while shallow copy only copies the parent object.
