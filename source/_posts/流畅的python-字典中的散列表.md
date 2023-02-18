---
title: 流畅的python--字典中的散列表
date: 2017-06-15 22:27:44
tags:
    - python
---
字典中的散列表其实就是稀疏数组（总会有一些元素是空白的数组），散列表中的单元叫：表元，在构建字典时会有产生两个表元，一个用来标记健，一个用来标记值，因为两个表元的长度都是相等的，所以可以通过表元的偏移量来查找字典。
python中最具有效率的内置数据类型就是字典和集合，这两种都是通过散列表来实现的。为了测试python中字典，集合和其他的数据类型的速度我做了以下的实验：
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
``````
结果：
```
list : 0.012472
set : 0.000079
dict : 0.000071
```

从上面可以看出集合与字典相比其他类型要快得多，这是因为列表中没有散列表支持in运算符。
散列的算法：比如python要获取dict[key]背后的值，python会先使用hash(key)来计算key的散列值，用然后去散列值的最低位的几位数字作为表元的偏移量，如果表元是空的，就会抛出一个KeyError异常，如果找到，就会获得一个:key:value值，python再最后会比较两个key的散列值是否一致。如果为真返回value，如果不相等这时候就称为散列冲突。这种情况的发生是因为散列表把随机的元素映射到几位数字上去。而散列表有依赖这个数字做检索。为了解决这个问题，散列表就会再从新取几位数字，去表元中寻找。如果表元为空这回抛出一个keyerror异常，如果没有找到就会重复上面的步骤直到找到。
