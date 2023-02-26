---
title: 流畅的python--字典中的散列表
date: 2017-06-15 22:27:44
tags:
    - python
---
字典中的散列表其实就是稀疏数组，也就是说有些元素是空的。在构建字典时，会有两个表元，一个用来标记键，一个用来标记值。因为两个表元的长度相等，所以可以通过表元的偏移量来查找字典。

在Python中，最有效率的内置数据类型是字典和集合，这两种数据类型都是通过散列表实现的。为了测试Python中不同数据类型的速度，我进行了以下实验：

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
    print name,':','%f' % (time.clock() - start)

testTime(list_a, 'list')
testTime(set_a, 'set')
testTime(dict_a, 'dict')

```

结果如下：

```
list : 0.012472
set : 0.000079
dict : 0.000071

```

从上面可以看出，集合与字典相比其他类型要快得多，这是因为列表中没有散列表支持in运算符。

散列的算法是这样的：比如Python要获取`dict[key]`背后的值，Python会先使用`hash(key)`来计算key的散列值，然后取散列值的最低位的几位数字作为表元的偏移量。如果表元为空，就会抛出一个`KeyError`异常；如果找到，就会获得一个`key:value`值，Python再最后会比较两个key的散列值是否一致。如果为真返回value，如果不相等则称为散列冲突。这种情况的发生是因为散列表把随机的元素映射到几位数字上去，而散列表需要依赖这个数字做检索。为了解决这个问题，散列表会再从新取几位数字，去表元中寻找。如果表元为空，就会抛出一个`KeyError`异常；如果没有找到，就会重复上面的步骤直到找到。
