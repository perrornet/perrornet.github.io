---
title: python dict
date: 2023-02-18 17:47:55
tags:
    - python 
---
Python's `dict` is a mapping type that uses a hash table to store data, making the query speed time complexity O(1), which is a typical data structure that trades space for time.

When we create a `key-value` pair, we first apply the hash function to the `key` to obtain an integer array, take the remainder of this integer with the length of the array that stores the `value`, and obtain the index where the `value` is stored. When initializing `dict`, Python first allocates an array of about 8KB to store `value` (see the definition of the minimum capacity of the dictionary at line 111 of `dictobject.c`). If the position of the array is not enough to store it, the system will be asked to allocate an array twice the current capacity to store `value`.

For hash conflicts, Python chose to use open addressing to solve the problem: when a hash conflict occurs, the next candidate position is calculated through the probing function. If the next candidate position still has a conflict, continue to search down through the probing function until an empty slot is found to store the element to be inserted.

`dict` initialization is completed in the function [dictobject.PyDict_New:691](https://github.com/python/cpython/blob/main/Objects/dictobject.c#L691):

```
PyObject * PyDict_New(void)
{
    dictkeys_incref(Py_EMPTY_KEYS);
    return new_dict(Py_EMPTY_KEYS, empty_values);
}

```

Since Python 3.6, the order of the keys in `dict` has been maintained in the order of insertion [dictobject.c:13](https://github.com/python/cpython/blob/main/Objects/dictobject.c#L13):

```
As of Python 3.6, this is compact and ordered. Basic idea is described here:
* <https://mail.python.org/pipermail/python-dev/2012-December/123028.html>
* <https://morepypy.blogspot.com/2015/01/faster-more-memory-efficient-and-more.html>
```

The implementation of `dict` and the method of solving hash conflicts are core issues of Python. Understanding these contents is of great help for Python's performance optimization and underlying implementation.
