---
title: Hash tables in Python dictionaries
date: 2017-06-15 22:27:44
tags:
    - python
---
# 

Dictionaries in Python use hash tables, which are actually sparse arrays, meaning that some elements are empty. When constructing a dictionary, two table elements are used, one to mark the keys and the other to mark the values. Because the lengths of the two table elements are equal, the dictionary can be searched by the offset of the table elements.

In Python, the most efficient built-in data types are dictionaries and sets, both of which are implemented through hash tables. To test the speed of different data types in Python, I conducted the following experiment:

```
import time
MAX = 10000000
list_a = [i for i in range(MAX)]
set_a = set(list_a)
dict_a = {}.fromkeys(list_a)

test = [i for i in range(1000)]

def testTime(x,name):
    start = time.clock()
    for i in test:
        if i in x:
            pass
    print(name,':','%f' % (time.clock() - start))

testTime(list_a, 'list')
testTime(set_a, 'set')
testTime(dict_a, 'dict')

```

The results are as follows:

```
list : 0.012472
set : 0.000079
dict : 0.000071

```

From the above, it can be seen that sets and dictionaries are much faster than other types. This is because lists do not support the `in` operator with hash tables.

The hash algorithm works as follows: for example, if Python wants to obtain the value behind `dict[key]`, it first uses `hash(key)` to calculate the hash value of the key, and then takes the lowest few digits of the hash value as the offset of the table element. If the table element is empty, a `KeyError` exception will be thrown; if found, a `key:value` value will be obtained, and Python will compare the hash values of the two keys. If they are identical, the value is returned, and if they are not equal, it is called a hash collision. This occurs because the hash table maps random elements to a few digits, and the hash table needs to rely on this number for retrieval. To solve this problem, the hash table will take a few more digits and search the table elements. If the table element is empty, a `KeyError` exception will be thrown; if not found, the above steps will be repeated until found.
