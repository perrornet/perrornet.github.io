---
title: python dict
date: 2023-02-18 17:47:55
tags:
    - python 
---
Python的`dict`是一种映射类型，底层使用哈希表的方式来储存数据，因此`dict`的查询速度时间复杂度为O(1)，这是一种典型的空间换时间的数据结构。

当我们创建一个`key-value`时，首先将哈希函数作用于`key`上获得一个整型数组，将这个整型数字与储存`value`数组的长度取余，得到该数组的下标，该下标就是`value`所存放的位置。在初始化`dict`时，Python首先会申请一个大小为8KB左右的数组用来存放`value`（见`dictobject.c`第111行定义字典最小容量），如果数组位置不够存放则会再向系统申请两倍于当前容量的数组来存放`value`。

对于哈希冲突，Python选择了开放寻址法来解决：当产生哈希冲突时，通过探测函数计算出下一个候选位置，如果下一个候选位置还是有冲突，那么不断通过探测函数往下找，直到找个一个空槽来存放待插入元素。

`dict`的初始化在[dictobject.PyDict_New:691](https://github.com/python/cpython/blob/main/Objects/dictobject.c#L691)该函数中完成：

```
PyObject * PyDict_New(void)
{
    dictkeys_incref(Py_EMPTY_KEYS);
    return new_dict(Py_EMPTY_KEYS, empty_values);
}

```

自Python 3.6以来，`dict`的键的顺序保持了插入时的顺序[dictobject.c:13](https://github.com/python/cpython/blob/main/Objects/dictobject.c#L13)

```
As of Python 3.6, this is compact and ordered. Basic idea is described here:
* <https://mail.python.org/pipermail/python-dev/2012-December/123028.html>
* <https://morepypy.blogspot.com/2015/01/faster-more-memory-efficient-and-more.html>

```

`dict`的实现方式以及哈希冲突的解决方法都是Python的核心问题。了解这些内容对于Python的性能优化和底层实现有很大帮助。
