---
title: python的高级用法 -- namedtuple
date: 2017-06-07 23:47:00
tags:
---
`namedtuple` 函数可以用来创建一个有名字的元组或者类，这种方法可以更有效的调试代码。下面是一个例子：

```
from collections import namedtuple

# 创建一个类
a = namedtuple('c', ('name', 'age'))
x = a('perror', 21)

# 该返回值可以用访问对象的方法来访问
print(x.name)
print(x.age)

# 也可以用标的方式去访问
print(x[0])

```

除了继承普通元组的方法外，命名元组还有额外的三个方法：`_fields` 类属性，`make()` 类方法，`_asdict()` 类实例方法。

下面是这些方法的具体用法：

```
# 查看该命名元组中的所有字段
print(x._fields)

# `_make` 通过一个可迭代对象来生成一个类实例
s = ('perror', '21')
new_s = x._make(s)

# 通过类实例方法更友好的打印出元组的字段
print(new_s._asdict())

```

注意，命名元组创建后，可以用其属性名或标的方式来访问其中的元素。
