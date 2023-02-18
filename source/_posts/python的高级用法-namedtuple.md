---
title: python的高级用法 -- namedtuple
date: 2017-06-07 23:47:00
tags:
---
namedtuple函数用来构建一个有名字的元组或者类，这种方法可以更有效的调试
代码：
```
>>> from collections import namedtuple
# 创建一个类
>>> a = namedtuple('c', ('name', 'age'))
>>> x = a('perror', 21)
>>> print x
c(name='perror', age=21)
>>> # 该返回值可以用访问对象的方法来访问
>>> print x.name
perror
>>> print x.age
21
>>> # 也可以用标的方式去访问
>>> print x[0]
perror
```
命名元组除了继承普通元组的方法外还有额外的三个方法：_fields 类属性，make() 类方法，_asdict() 类实例方法
```
>>> # 查看该命名元组中的所有字段
>>> x._fields
('name', 'age')
>>> # _make 通过一个可迭代对象来生成一个类实例
>>>s = ('perror', '21')
>>> new_s = x._make(s)
>>> # 通过类实例方法更友好的打印出元组的字段
>>>new_s._asdict()
OrderedDict([('name', 'perror'), ('age', 23)])
```