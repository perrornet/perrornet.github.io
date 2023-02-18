---
title: python dict
date: 2023-02-18 17:47:55
tags:
    - python 
---
python的dict是一种映射类型, 底层使用了哈希表的方式来储存数据, 因此dict的查询速度的时间复杂度为O(1),这是一种典型的空间换时间的数据结构。

当我们创建一个key-value时，首先将哈希函数作用于key上获得一个整型数组，将这个整型数字与储存value数组的长度取余， 得到该数组的下标， 该下标就是value所存放的位置。 python在初始化dict时首先会申请一个大小为8KB左右的数组用来存放value（见dictobject.c第111行定义字典最小容量） ，如果数组位置不够存放则会再向系统申请两倍于当前容量的数组来存放value。

对于哈希冲突python选择了开放寻址法来解决：当产生哈希冲突时，通过探测函数计算出下一个候选位置，如果下一个候选位置还是有冲突，那么不断通过探测函数往下找，直到找个一个空槽来存放待插入元素。

dict的初始化在[dictobject.PyDict_New:691](https://github.com/python/cpython/blob/main/Objects/dictobject.c#L691)该函数中完成：

```
PyObject * PyDict_New(void)
{
dictkeys_incref(Py_EMPTY_KEYS);
return new_dict(Py_EMPTY_KEYS, empty_values);
}
```
在python3.6之后dict的键的顺序保持了插入时的顺序 [dictobject.c:13](https://github.com/python/cpython/blob/main/Objects/dictobject.c#L13)

```
As of Python 3.6, this is compact and ordered. Basic idea is described here:
* https://mail.python.org/pipermail/python-dev/2012-December/123028.html
* https://morepypy.blogspot.com/2015/01/faster-more-memory-efficient-and-more.html
```