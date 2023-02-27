---
title: Advanced Usage of Python -- namedtuple
date: 2017-06-07 23:47:00
tags:
---
The `namedtuple` function can be used to create a named tuple or class, which can be more effective for debugging code. Here's an example:

```
from collections import namedtuple

# Create a class
a = namedtuple('c', ('name', 'age'))
x = a('perror', 21)

# Access via object method
print(x.name)
print(x.age)

# Access via indexing
print(x[0])

```

In addition to inheriting methods from normal tuples, named tuples have three additional methods: the `_fields` class attribute, `make()` class method, and `_asdict()` class instance method.

Here's how to use these methods:

```
# View all fields in the named tuple
print(x._fields)

# `_make` generates a class instance using an iterable object
s = ('perror', '21')
new_s = x._make(s)

# Print the tuple's fields in a more friendly format using the class instance method
print(new_s._asdict())

```

Note that after the named tuple is created, its elements can be accessed either by attribute name or by indexing.
