---
title: python内置函数
date: 2017-12-15 17:21:05
tags:
    - python
    - 技术
---
abs(x): 接受一个整数或浮点数作为参数，返回该参数的绝对值，如果该参数是一个复数，则返回复数的模。

```
>>> abs(-1)
1
>>> abs(4 - 8j)
8.94427190999916
```
all(iter): 接受一个可迭代类型，如果该参数内所有元素为真返回True，否则返回False。

```
>>> all([1,2,3,4,5])
True
>>> all([1,2,3,4,0])
False
```
any(iter):接受一个可迭代类型为参数,如果该参数内任何元素为真,则返回True,否则返回False.

```
>>> any([0, 0,1])
True
>>> any([0, 0,0])
False
```
asicc(object):接受一个对象, 将调用该对象的__repr__方法, 返回的字符串会将\x,\u,\U进行转移输出.


```
In [4]: class a:
...:     def __repr__(self):
...:         return "repr"
In [5]: ascii(a())
Out[5]: 'repr'
In [9]: ascii("\n\rfdsafdsa")
Out[9]: "'\\n\\rfdsafdsa'"
```
bin(x):接受一个整数作为参数, 并将该参数转换成以"0b"前缀的二进制字符串,如果x不是一个整数,那么将会调用对象的__index__方法的整数并转换成二进制.


```
In [15]: class a:
...:     def __index__(self):
...:         return 1
In [14]: bin(1)
Out[14]: '0b1'
In [16]: bin(a())
Out[16]: '0b1'
```
bool([x]):如果不传参数,则默认返回Fase,如果X是一个不是整数或浮点数则首先调用对象的__bool__方法, 如果对象的__bool__方法未定义,其次调用__len__方法,如果返回0则bool()返回False,否则返回True,如果对象__len__、__bool__方法都未定义则默认返回True


```
In [39]: bool(1)
Out[39]: True
In [40]: bool(0.0)
Out[40]: False
In [41]: bool(0 - 0j)
Out[41]: False
In [43]: class a:
...:     def __bool__(self): return True
...:     def __len__(self): return 0
In [45]: bool(a())
Out[45]: True
In [44]: bool(a)
Out[44]: True
```
bytearray([source[, encoding[, errors]]]):接受一个大于等于0小于256的整数或者一个可迭代类型、如果参数一个字符串,那么必须指定encoding. 返回一个新的字节数组.如果传入一个对象首先调用该对象__index__, 如果未定义__index__则调用对象__iter__方法.


```
In [64]: class a:
...:     def __iter__(self):
...:         for i in range(0, 256):
...:             yield i
...:     def __index__(self): return 1
...:
In [65]: bytearray(a())
Out[65]: bytearray(b'\x00')
In [48]: bytearray()
Out[48]: bytearray(b'')
In [49]: bytearray([1,2,3,4,5,6,7])
Out[49]: bytearray(b'\x01\x02\x03\x04\x05\x06\x07')
In [77]: bytearray(1)
Out[77]: bytearray(b'\x00')
```
bytes([source[, encoding[, errors]]]): 如同bytearray使用方法, 不过该函数返回的一个不可变类型.


```
In [72]: class a:
...:     def __iter__(self):
...:         for i in range(0, 256):
...:             yield i
...:     def __index__(self): return 1
...:
In [73]: bytes(a())
Out[73]: b'\x00'
In [79]: bytes(1)
Out[79]: b'\x00'
In [80]: bytes()
Out[80]: b''
In [81]: bytes([1,2,3,4,5,6,7])
Out[81]: b'\x01\x02\x03\x04\x05\x06\x07'
```
callable(object):接受一个对象作为参数, 如果该参数是可以调用的则返回True, 否则返回False

```
In [83]: class a:pass
In [85]: callable(a)
Out[85]: True
In [86]: callable(a())
Out[86]: False
In [88]: class a:
...:     def __call__(self):
...:         return 1

In [89]: callable(a())
Out[89]: True
```
chr(i): 接受一个整数, 返回该整数对应的Unicode代码点的字符串,该整数必须大于等于0,小于等于1114112.

```
In [105]: chr(0)
Out[105]: '\x00'
In [99]: chr(1114111)
Out[99]: '\U0010ffff'
In [100]: chr(1114112) # 超出范围
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-100-4857faf08086> in <module>
----> 1 chr(1114112)

ValueError: chr() arg not in range(0x110000)==
```
@classmethod:装饰器, 作用于类方法上, 被装饰的方法将转换成类方法,类方法的一个参数将是类本身.

```
In [110]: class a:
...:     def a1(self):
...:         print(self)
...:     @classmethod
...:     def a2(self): # 按照约定应该将self修改成cls来从字面量上区分类方法和实例方法.
...:         print(self)
...:

In [111]: a().a1() # 实例方法
<__main__.a object at 0x00000245CBE815C0>
In [112]: a.a2() # 类方法
<class '__main__.a'>
In [113]: a().a2() # 实例可以调用类方法
<class '__main__.a'>
In [115]: a.a1() # 类在未实例化之前不能调用实例化方法, 当然有捷径可以走,这里只是做了一个区分, 不展示捷径
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-115-2656506098d9> in <module>
----> 1 a.a1()

TypeError: a1() missing 1 required positional argument: 'self'
```
compile(source, filename, mode, flags=0, dont_inherit=False, optimize=-1):将一个字符串编译成字节代码:
```
source: 字符串或者AST对象,
filename: 代码文件名称，如果不是从文件读取代码则传递一些可辨认的值, 如’’.
mode: 指定编译代码的种类。可以指定为 exec, eval, single
flags: 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。
flags和dont_inherit是用来控制编译源码时的标志
In [121]: d = compile("print(1)", "", "exec")
In [122]: d
Out[122]: <code object <module> at 0x00000245CBF448A0, file "", line 1>
In [123]: exec(d)
1
complex([real[, imag]]): 创建一个值为 real + imag * j 的复数或者转化一个字符串或数为复数。如果第一个参数为字符串，则不需要指定第二个参数。


```
In [125]: complex(1, 2)
Out[125]: (1+2j)
In [127]: complex("1")
Out[127]: (1+0j)
In [128]: complex("1+2j") # 此处不能有空格, 否则会报错
Out[128]: (1+2j)
delattr(object, name): 接受一个对象以及一个字符串,删除该对象的name属性,也可以使用del object.name来实现.

```
In [144]: class a:
...:     x = 1
...:     y = 2
...:
In [145]: delattr(a, "x")
In [147]: a.x
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-147-e27186f28a17> in <module>
----> 1 a.x
AttributeError: type object 'a' has no attribute 'x'

In [148]: a.y
Out[148]: 2
```
dict([object]):用于创建一个字典.如果不传参数,则返回一个空的字典.
```
In [150]: dict()
Out[150]: {}
In [151]: dict(a=1, b=2)
Out[151]: {'a': 1, 'b': 2}
In [152]: dict({"a":1})
Out[152]: {'a': 1}
``` 
dir([object]):不传参数时, 返回当前范围内变量、方法和定义的类型列表,传入参数时返回该参数的所有的属性、方法, 如果该参数定义了__dir__方法, 则使用__dir__返回值, 否则该函数将尽可能的最大限度的收集该对象的方法以及属性.

```
In [157]: dir("")
Out[157]:
['__add__',
'__class__',
'__contains__',
...]
In [155]: class a:
...:     def __dir__(self):
...:         return ["a", "b"]
...:
In [156]: dir(a())
Out[156]: ['a', 'b']
In [158]: class a:
...:     x = 1
...:     y = 2
...:
In [159]: dir(a)
[...,
'x',
'y']
```
divmod(x, y): 接受两个非复数的数字作为参数, 返回X,Y的商和余.
```
In [160]: divmod(1, 2)
Out[160]: (0, 1)
In [161]: divmod(1.5, 2)
Out[161]: (0.0, 1.5)
```
enumerate(iterable, start=0):接受一个可迭代类型以及一个可选int类型的参数, 返回一个枚举对象.如果指定了start则该可迭代对象的下边从start开始.

```
In [163]: enumerate([1,2,3,4])
Out[163]: <enumerate at 0x245cbf50438>
In [164]: list(enumerate([1,2,3,4]))
Out[164]: [(0, 1), (1, 2), (2, 3), (3, 4)]
```
eval(expression, globals=None, locals=None):接受一个字符串类型的表达式, 并执行该表达式,返回其值.

```
expression: 字符串类型的表达式.
globals: 字典类型, 变量作用域，全局命名空间.
locals: 任何的映射对象, 变量作用域，局部命名空间.
In [166]: eval("1 + 2")
Out[166]: 3
exec(object[, globals[, locals]]):该函数提供动态执行python代码的功能,object可以是字符串也可以是一个代码对象. globals必须是一个字典,locals则可以是任何的映射对象,

In [168]: x = 1
In [169]: exec("x + 2")
In [172]: x
Out[172]: 3
```
filter(function,iterable):接受一个函数对象以及一个可迭代对象作为参数, 将可迭代对象中的每一个元素传入第一个参数中,如果该函数返回True则保留当前元素,否则删除当前元素.filter函数返回一个可迭代对象.
```
In [3]: list(filter(lambda x: False, [1,2,3,4]))
Out[3]: []
In [4]: list(filter(lambda x: True, [1,2,3,4]))
Out[4]: [1, 2, 3, 4]
```
float([x]): 接受一个可选整型、浮点型、对象或者字符串类型参数, 返回该参数的浮点类型数据,如果X是一个自定义对象那么将调用对象的__float__方法(仅提供用方法, 完整方法见:float).

```
In [6]: float("+1")
Out[6]: 1.0

In [7]: float("-1")
Out[7]: -1.0

In [8]: float("Inf")
Out[8]: inf
In [11]: float(1)
Out[11]: 1.0
In [12]: float(1.2)
Out[12]: 1.2
# 自定义对象
In [14]: class a:
...:     def __float__(self): return 1.0
...:
In [15]: float(a())
Out[15]: 1.0
```
format(value[,format_spec]):格式化字符串函数, 接受不限个参数，位置可以不按顺序, 其数字格式化符号见菜鸟教程, 对于自定义对象, 则调用对象的__format__方法, 如果对象没有定义__format__则使用对象__str__方法.

```
In [20]: "{}|{}".format(1, "dsa")
Out[20]: '1|dsa'

In [21]: "{1}|{0}".format(1, "dsa")
Out[21]: 'dsa|1'

In [22]: "{0}|{1}".format(1, "dsa")
Out[22]: '1|dsa'

In [23]: "{:.2f}".format(1.235646)
Out[23]: '1.24'

In [33]: class a:
...:     def __format__(self, *args):
...:         return "format"
...:
In [34]: "{}".format(a())
Out[34]: 'format'

In [36]: class a:
...:     def __str__(self):
...:         return "str"
...:
In [37]: "{}".format(a())
Out[37]: 'str'
```
frozenset([iterable]): 接受一个可选的可迭代类型的参数, 返回一个frozenset对象, 该对象与set对象基本相似, 但是该对象是不可变的.

```
In [39]: frozenset()
Out[39]: frozenset()

In [40]: frozenset([1,2,1,1])
Out[40]: frozenset({1, 2})
```
getattr(object,name[,default]): 接受两个必选参数和一个可选参数, 该函数返回了object.name的值, 如果不存在name属性,并且没有传入default参数时则触发异常, 如果传入了default参数则没有找到name属性时返回default.

```
In [45]: class a:
...:     def b(self):
...:         return "2"

In [47]: getattr(a(), "b")()
Out[47]: '2'

getattr(a(), "b1", lambda : print("b1"))()
b1
```
globals(): 返回当前所有全局变量的字典.
```
In [49]: globals()

Out[49]:
{'__name__': '__main__',
'__doc__': 'Automatically created module for IPython interactive environment',
'__package__': None,
...
}
```
hasattr(object, name): 接受两个必选参数, 检测object中是否有那么属性.如果有name属性返回True,否则返回False.

```
In [50]: class a:
...:     b = 1
...:

In [51]: hasattr(a, "b")
Out[51]: True

In [52]: hasattr(a, "b1")
Out[52]: False
```
hash(object): 接受一个必选参数, 返回该对象的整型哈希值, 如果时自定义对象则调用对象的__hash__方法.

```
In [56]: hash("fdsaf")
Out[56]: 2111967917974353823

In [58]: class a:
...:     def __hash__(self):
...:         return 9527
...:
In [59]: hash(a())
Out[59]: 9527
```
help([object]): 接受一个可选的参数, 该参数可以是任意的类型, 返回该对象的帮助文档, 如果是自定义对象, 则返回该函数或者类的注释.如果没有传入参数则拉起交互式的帮助系统

```
In [62]: help(1)
Help on int object:
class int(object)
|  int(x=0) -> integer
...

In [65]: class a:
...:     """这是help信息"""
...:     def b(self):
...:         """这也是help信息"""
...:         pass

In [66]: help(a())
Help on a in module __main__ object:
class a(builtins.object)
|  这是help信息
|
|  Methods defined here:
|
|  b(self)
|      这也是help信息
|
|  ----------------------------------------------------------------------
|  Data descriptors defined here:
|
|  __dict__
|      dictionary for instance variables (if defined)
|
|  __weakref__
|      list of weak references to the object (if defined)
```
hex(x):接受一个整型作为必选参数, 返回该参数的16进制字符串, 该字符串以0x开头,如果x是一个自定义对象那么该对象必须实现__index__方法.

```
In [72]: hex(185)
Out[72]: '0xb9'

In [74]: class a:
...:     def __index__(self): return 1
In [75]: hex(a())
Out[75]: '0x1'
```
id(object): 返回该对象在内存中的地址以整型返回.

```
In [77]: id(1)
Out[77]: 1786755536
In [81]: class a:
...:     def __init__(self, x):
...:         self.x = x
...:
In [82]: id(a(1)) == id(2)
Out[82]: False
```
input([prompt]): 从标准输入中读取输入一回车键结束, 如果传入prompt, 则将prompt打印到标准输出中.

```
In [84]: input("这是提示:")
这是提示:dd
Out[84]: 'dd'
```
int(x, base=10):将一个字符串或数字转换为整型,如果没有传入参数则返回0,如果传入传入了base则表示X是base进制的数字.如果X是一个自定义对象, 则调用__init__方法.

```
In [86]: int()
Out[86]: 0

In [87]: int(1.0)
Out[87]: 1

In [90]: int("0x9b", 16)
Out[90]: 155

In [91]: class a:
...:     def __int__(self):
...:         return 1
In [92]: int(a())
Out[92]: 1
```
isinstance(object, classinfo):如果object是classinfo的实例或者子类的实例, 返回True, 否则返回False.如果classinfo是一个元组, 那么object只要是元组中任意一个元素的子类的实例或者实例化都返回True.

```
In [94]: class a: pass
In [95]: class b(a): pass

In [99]: isinstance(b(), a)
Out[99]: True

In [101]: isinstance(1, int)
Out[101]: True

In [103]: isinstance(1, str)
Out[103]: False
```
issubclass(class, classinfo):如果class是classinfo的子类返回true, 使用方法与isinstance相同.

```
In [94]: class a: pass
In [95]: class b(a): pass

In [105]: issubclass(b, a)
Out[105]: True
```
iter(object[,sentinel]):返回一个迭代器对象, 如果没有传入第二个参数, 那么object必须是实现了迭代协议或者序列协议否则将触发TypeError, 如果传入了sentinel参数那么object必须是可调用对象, 此时每迭代一次都调用object,如果object返回的值等于sentinel时,触发StopIteration异常.

```
In [108]: iter([1,2,3])
Out[108]: <list_iterator at 0x210bde35f98>

In [110]: class a:
...:     def __iter__(self):
...:         for i in range(10):
...:             yield i
...:

In [111]: list(iter(a()))
Out[111]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [12]: def a():
...:     global num
...:     num += 1
...:     if num >= 10:
...:         return -1
...:     return num

In [13]: for i in iter(a, -1):
...:     print(i)
1
2
3
4
5
6
7
8
9
```
len(s): 接受一个序列或者集合类型对象, 返回其长度, 如果时自定义对象, 将会调用对象的__len__方法.

```
In [18]: len([1,2,3])
Out[18]: 3

In [19]: len({1:2, 3:4})
Out[19]: 2

In [20]: class a:
...:     def __len__(self): return 1

In [21]: len(a())
Out[21]: 1
```
locals():返回当前本地和只有变量的值.

```
In [24]: def a():
...:     x = 1
...:     print(locals())
In [25]: a()
{'x': 1}
```
map(function,iterable…):返回一个迭代器, 该函数将function应用与每一个iterable元素上并返回直到可迭代对象耗尽.

```
In [27]: map(lambda x: x ** 2, [1,2,3])
Out[27]: <map at 0x2aec3ae4a20>

In [28]: list(map(lambda x: x ** 2, [1,2,3]))
Out[28]: [1, 4, 9]
```
max(arg1,arg2,*args[,key]):接受多个位置参数或者一个可迭代对象, 返回其中最大的值.

```
In [39]: max(1,2)
Out[39]: 2

In [40]: max([2,3,4,5])
Out[40]: 5
```
memoryview(obj): 返回obj的内存查看对象(对支持缓冲区协议的数据进行包装，在不需要复制对象基础上允许Python代码访问).

```
In [43]: a = b"dsadsad"
In [44]: d = memoryview(a)

In [47]: d[0]
Out[47]: 10
# 未复制数据
In [52]: id(d.obj)
Out[52]: 2949631565952
In [53]: id(a)
Out[53]: 2949631565952
```
min(arg1，arg2，* args[，key]): 接受多个位置参数或者一个可迭代对象, 返回其中最小的值.

```
In [55]: min(1,2)
Out[55]: 1

In [56]: min([1,2,3,5])
Out[56]: 1
```
next(iterator[,default]): 通过调用iterator的__next__方法并返回其值.

```
In [58]: class a:
...:     def __next__(self):
...:         for i in range(10):
...:             yield i

In [60]: list(next(a()))
Out[60]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
object(): 返回一个新的无任何特征的对象, object时所有class的基类.

```
In [105]: object()
Out[105]: <object at 0x2aec3a34880>

In [106]: class a:pass
In [107]: issubclass(a, object)
Out[107]: True
```
oct(x):接受一个整型作为参数返回以"0o"为前缀的八进制字符串,如果x是自定义对象, 则调用该对象的__index__方法.

```
In [109]: oct(1)
Out[109]: '0o1'

In [111]: class a:
...:     def __index__(self): return 1

In [112]: oct(a())
Out[112]: '0o1'
```
open(file,mode='r’,buffering=-1,encoding=None,errors=None,newline=None,
closefd=True,opener=None):打开file文件并返回一个文件对象, 具体参数值见open.

```
file: 文件所在的路径以及文件名
mode: 打开文件的模式, 默认"r”
buffering: 用于设置缓冲策略, 0为关闭缓冲, 1为选择行缓冲, > 1以指示固定大小的块缓存区.默认-1,二进制文件以固定大小的块缓冲 4.encoding:用于解码或编码文件的编码的名称, 默认编码取决于所在的操作系统 5.errors:指定如何处理编码和解码错误, 仅能在文本模式下使用 6.newline: 控制通用换行模式的工作方式（仅适用于文本模式）。它可以是None，''，'\n’，'\r’，和’\r\n’
In [114]: open("test.test", "w").write("fdsa")
Out[114]: 4
```
ord(c): 接受一个字符串类型参数, 返回该Unicode代码点的整数.
```
In [116]: ord("a")
Out[116]: 97

In [117]: ord("你")
Out[117]: 20320
```
pow(x,y[,z]):当未传入Z参数时,返回x的y次方,如果传入了Z则对结果取余.
```
In [122]: pow(2,3)
Out[122]: 8

In [124]: pow(2,3, 2)
Out[124]: 0
```
print(*objects,sep=’ ‘,end=’\n’,file=sys.stdout,flush=False):将objects对象打印到标准输出中,每一个objects使用sep分割, 以end结束,对于自定义对象, 首先调用对象__str__方法, 如果没有定义__str__方法则调用__repr__方法.

```
In [136]: class a:
...:     def __str__(self): return "str"
...:     def __repr__(self): return "repr"

In [137]: print(a())
str
```
property(fget=None，fset=None，fdel=None，doc=None): 可作为装饰器使用也可作为函数使用.用于显示类属性的读取、设置以及删除. 作为函数使用时:

```
In [140]: class a:
...:     def __init__(self):
...:         self._x = None
...:
...:     def getx(self):
...:         return self._x
...:
...:     def setx(self, x):
...:         self._x = x
...:     def delx(self):
...:         self._x = None
...:     x = property(getx, setx, delx, "test")
```
作为装饰器使用时:
```
class a:
def __init__(self):
self._x = None
@property
def x(self):
return self._x
@x.fset
def x(self, x):
self._x = x
@x.fdel
def x(self):
self._x = None
参数:

fget – 获取属性值的函数
fset – 设置属性值的函数
fdel – 删除属性值函数
doc – 属性描述信息
```
range(start,stop[,step]):range并不是一个函数, range时一个不可变序列类型,返回一个迭代器对象.

```
In [149]: list(range(1, 10, 2))
Out[149]: [1, 3, 5, 7, 9]

In [150]: list(range(1, 10))
Out[150]: [1, 2, 3, 4, 5, 6, 7, 8, 9]

In [151]: list(range(10))
Out[151]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [152]: type(range) # 非函数
Out[152]: type
```
repr(object): 返回可打印的字符串,此函数尝试返回一个字符串，该字符串在传递时会产生具有相同值的对象eval()，否则表示形式是一个括在尖括号中的字符串，其中包含对象类型的名称以及其他信息通常包括对象的名称和地址。类可以通过定义__repr__()方法来控制此函数为其实例返回的内容.

```
In [154]: repr(1)
Out[154]: '1'

In [155]: class a:
...:     def __repr__(self): return "1"

In [156]: repr(a())
Out[156]: '1'

In [157]: repr(a)
Out[157]: "<class '__main__.a'>"
```
reversed(seq):接受一个可迭代类型, 返回该可迭代类型的反向迭代器.

```
In [161]: list(reversed(range(10)))
Out[161]: [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```
round(num, [, ndigits]): 返回num四舍五入精确到ndigits精度数字, 如果没有传入ndigits则返回最接近num的整数, 如果num时一个自定义对象, 则调用__round__方法.

```
In [163]: round(1.2563, 2)
Out[163]: 1.26

In [164]: round(1.2563)
Out[164]: 1
```
set([iterable]):返回一个新的set类型对象, 接受可选可迭代类型参数, 并将可迭代类型参数转换成set类型返回.

```
In [166]: set()
Out[166]: set()

In [167]: set([1,2,3])
Out[167]: {1, 2, 3}

In [168]: set(range(10))
Out[168]: {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```
setattr(object, name, value): 接受三个必选参数, 将object.name属性赋值为value.

```
In [172]: class a: pass
In [173]: a1 = a()
In [174]: setattr(a1, "b", 1)
In [175]: a1.b
Out[175]: 1
```
slice(start,stop[,step]):返回表示由指定的索引集的切片对象, 该对象为只读数据属性.

```
In [178]: slice(0, 1)
Out[178]: slice(0, 1, None)
```
sorted(iterable,*,key=None,reverse=False):将_iterable_对象排序后返回.
```
key: 可选关键字参数,_key_指定一个参数的函数，该函数用于从_iterable中的_每个元素中提取比较键
reverse: 可选关键字参数, 如果设置为True，则列表元素将按照每个比较相反的方式进行排序.

In [181]: sorted(["12", "1", "34", "6", "90"], key=int)
Out[181]: ['1', '6', '12', '34', '90']

In [182]: sorted(["12", "1", "34", "6", "90"], key=int, reverse=True)
Out[182]: ['90', '34', '12', '6', '1']
```
@staticmethod: 装饰器函数, 作用于类方法上, 将方法转换成静态方法.

```
In [185]: class a:
...:     @staticmethod
...:     def b(x):
...:         return x

In [186]: a.b(1)
Out[186]: 1
```
str(object=b’',encoding='utf-8’,errors='strict’):返回一个字符串类型的对象,如果object是一个自定义对象, 那么首先会调用对象的__str__方法, 如果没有实现__str__方法则调用__repr__方法.

```
In [194]: class a:
...:     def __repr__(self): return "re"
...:     def __str__(self): return "str"

In [195]: str(a())
Out[195]: 'str'
```
sum(iterable[,start]):如果没有传入_start_参数, 则将_iterable_所有的元素叠加后返回, 如果传入了_start_参数,则从_start_参数开始将_iterable_的元素从左至右的叠加.

```
In [201]: sum([1,2,3], 8)
Out[201]: 14

In [207]: sum(range(1, 101))
Out[207]: 5050
```
super([type[,object-or-type]]):该函数用于调用父类(超类)的一个方法.
```
In [209]: class a:
...:     def add(self, x):
...:         return x + 1

In [210]: class b(a):
...:     def add(self, x):
...:         return super().add(x)

In [211]: b().add(1)
Out[212]: 2
```
tuple([iterable]):返回一个元组对象, 接受一个可选的可迭代参数, 如果传入了该参数则将该参数转换成元组对象.

```
In [214]: tuple([1,2,3,4])
Out[214]: (1, 2, 3, 4)

In [215]: tuple()
Out[215]: ()
```
type(name,bases,dict):只传入一个参数时将返回该参数的类型,使用三个参数，返回一个新类型对象,name则成为类对象的名称,base则表明了其父类, 可选的关键字参数则将成为类的属性.(type是类型实例关系的顶端, object是父子关系的顶端，所有的数据类型的父类都是它, Object是type的一个实例,Type是object的子类, 详细见type和object之间的关系)

```
In [222]: d = type("objects", (), {"names": "this is name"})

In [223]: d
Out[223]: __main__.objects
In [224]: d.names
Out[224]: 'this is name'

In [217]: type(1)
Out[217]: int
```
vars([object]):返回模块、类、实例、其他对象的__dict__属性.

```
In [232]: class a:
...:     x = 1

In [233]: vars(a)
Out[233]:
mappingproxy({'__module__': '__main__',
'x': 1,
'__dict__': <attribute '__dict__' of 'a' objects>,
'__weakref__': <attribute '__weakref__' of 'a' objects>,
'__doc__': None})
```
zip(*iterables):将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表,如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同.

```
In [237]: zip([1,2,3,4], ["q", "w", "e", "r"])
Out[237]: <zip at 0x2aec3ad6d88>

In [238]: list(zip([1,2,3,4], ["q", "w", "e", "r"]))
Out[238]: [(1, 'q'), (2, 'w'), (3, 'e'), (4, 'r')]
```
import(name,globals=None,locals=None,fromlist=(),level=0):import 语句将会调用该函数,该函数导入模块_name_, 使用给定的_globals和_locals来确定如何解释包上下文中的名称.

```
In [240]: __import__("requests")

Out[240]: <module 'requests' from 'd:\\anaconda3\\envs\\python3\\lib\\site-packages\\requests\\__init__.py'>
In [241]: requests = __import__("requests")
In [242]: requests.__version__
Out[242]: '2.21.0'
```