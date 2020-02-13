---
layout: post
title:  "python内置对象说明"
date: 2019-3-01 08:15:00 +0800
categories: jekyll update
---
python内置对象说明
===
dir(__builtins__):打印所有内置对象，内置函数可以参考:[]!http://www.runoob.com/python/python-built-in-functions.html

* '__build_class__', build_class 函数是创建类对象过程中的一个核心函数

* '__debug__', 
只读变量,参考：[]!https://blog.csdn.net/you_are_my_dream/article/details/53328293

* '__doc__', 
每个对象都会有一个__doc__属性，用于描述该对象的作用。在一个模块被import时，其文件中的某些特殊的字符串会被python解释器保存在相应对象的__doc__属性中。比如，一个模块有模块的__doc__，一个class或function也有其对应的__doc__属性。在python中，一个模块其实就是一个.py文件。在文件中特殊的地方书写的字符串就是所谓的docstrings，就是将被放到__doc__的内容。这个“特殊的地方”包括：
1. 一个文件任何一条可执行的代码之前  #模块的__doc__
2. 一个类，在类定义语句后，任何可执行代码前#类的__doc__
3. 一个函数，在函数定义语句后，任何可执行代码前#函数的__doc__
```
#use  __doc__ 属性
class MyClass:
    'string.'
    def printSay():
        'print say welcome to you.'
        print 'say welcome to you.'
print MyClass.__doc__
print MyClass.printSay.__doc__
  
#输出结果
 string.
print say welcome to you.
```

* '__import__', 
import导入的是一个标准模块，而标准模块的概念是一个文件夹里面必须包含__init__.py文件。它的作用更像是一种声明，且import模块进来之后，万一原本的模块有什么变化，可以通过reload()进行重新加载。
__import__()作为一个函数，只能接受字符串参数，返回值可以直接用来操作，通常在动态加载的时候用到这个函数，最常见的情景就是插件功能的支持。

* '__loader__', 
[]!https://www.cnblogs.com/baishoujing/p/6358685.html 
'__loader__'是由加载器在导入的模块上设置的属性，访问它时将会返回加载器对象本身。

* '__name__', 
显示了当前模块执行过程中的名称，如果当前程序运行在这个模块中，__name__ 的名称就是__main__如果不是，则为这个模块的名称

* '__package__', 
获取导入文件的路径，多层目录以点分割，注意：对当前文件返回None

* '__spec__', TODO(找不到)

* 'abs', 
绝对值

* 'all', 
* 'any', 
all() # 全为真，输出Ture ， 则输出Flase
any() # 只要有真，输出Ture，则输出Flase
0,None,"",[],(),{}  表示False
```
n1 = all([1,2,3,[],None])
print (n1)
# False
n2 = any([1,0,"",[]])
print (n2)
# True
n1 = all([1,2,3,[],None])
print (n1)
# False
n2 = any([1,0,"",[]])
print (n2)
# True
```

* 'ascii', 
自动执行对象的__repr__方法

* 'bin', 
将十进制转为二进制  0b 表示二进制

* 'bool',
布尔值
0,None,"",[],(),{}  表示False

* 'breakpoint', 
Python 3.7添加了breakpoint()，这个内置函数使得函数被调用时，让执行切换到调试器。相应的调试器不一定是Python自己的pdb，可以是之前被设为首选调试器的任何调试器。以前，调试器不得不手动设置，然后调用，因而使代码更冗长。而有了breakpoint()，只需一个命令即可调用调试器，并且让设置调试器和调用调试器泾渭分明。

* 'bytearray', 
根据传入的参数创建一个新的字节数组
```
>>> bytearray('中文','utf-8')
bytearray(b'\xe4\xb8\xad\xe6\x96\x87')
```
* 'bytes',
根据传入的参数创建一个新的不可变字节数组
```
>>> bytes('中文','utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
``` 

* 'callable', 
检测传递的值是否可以调用
```
def f1():
    pass
f1()
f2 = 123
print (callable(f1))
# True
print (callable(f2))
# False
```

* 'chr', 
输出 ASCII的对应关系，chr()输出十进制位置的字符，ord()输出字符在ASCii表的位置
```
print (chr(65))
# A
print (ord("A"))
# 65
```

* 'classmethod', 
classmethod 是一个装饰器函数，用来标示一个方法为类方法。类方法的第一个参数是类对象参数，在方法被调用的时候自动将类对象传入，参数名称约定为cls。如果一个方法被标示为类方法，则该方法可被类对象调用(如 C.f())，也可以被类的实例对象调用(如 C().f())
```
>>> class C:
    @classmethod
    def f(cls,arg1):
        print(cls)
        print(arg1)

        
>>> C.f('类对象调用类方法')
<class '__main__.C'>
类对象调用类方法

>>> c = C()
>>> c.f('类实例对象调用类方法')
<class '__main__.C'>
类实例对象调用类方法
```

* 'compile', 
编译，将字符串编译成python代码，见exec

* 'complex', 
返回一个复数。有两个可选参数。当两个参数都不提供时，返回复数 0j。

* 'copyright', 
版权信息

* 'credits', 
感谢信息

* 'dict', 
用于创建一个字典

* 'dir', 
快速获取模块,对象提供的功能

* 'divmod',
得到商和余数

* 'enumerate',
多用于在for循环中得到计数，利用它可以同时获得索引和值，即需要index和value值的时候可以使用enumerate
```
#指定索引从1开始
>>> lst = [1,2,3,4,5,6]
>>> for index,value in enumerate(lst,1):
print ('%s,%s' % (index,value))
1,1
2,2
3,3
4,4
5,5
6,6
```

* 'eval',
执行表达式，并且获取结果
eval有返回值，exec没返回值

* 'exec',
执行python代码
eval有返回值，exec没返回值
```
s = "print('hello,python~')"
r = compile(s,"<string>","exec")
exec (r)
```

* 'exit', 
```sys.exit(n)``` 退出程序引发SystemExit异常，可以捕获异常执行些清理工作。n默认值为0，表示正常退出，其他都是非正常退出。还可以sys.exit(“sorry, goodbye!”); 一般主程序中使用此退出。
```os._exit(n)```， 直接退出, 不抛异常, 不执行相关清理工作。常用在子进程的退出。
```exit()/quit()```，跑出SystemExit异常。一般在交互式shell中退出时使用。

* 'filter',
函数返回值为Ture,将元素添加结果中,filter 循环第二个参数，让每个循环元素执行函数，如果函数返回值为Ture,表示函数合法
```
def f1(args):
    result = []
    for item in args:
        if item > 22:
            result.append(item)
    return result
li = [11,22,33,44,55,66,77,88]
ret = f1(li)
print (ret)
# [33, 44, 55, 66, 77, 88]
 
# 优化示例1
def f2(a):
    if a > 22:
        return True
ls1 = [11,22,33,44,55,66,77]
res1 = filter(f2,ls1)
print(list(res1))
# [33, 44, 55, 66, 77]
 
# 知识扩展，lambda 函数
res2 = filter(lambda x:x > 22,ls1)
print (res2) # 返回一个filter object
# <filter object at 0x000000E275771748>
print (list(res2))
# [33, 44, 55, 66, 77]
```

* 'float', 
函数用于将整数和字符串转换成浮点数

* 'format', 
函数功能将一个数值进行格式化显示。如果参数format_spec未提供，则和调用str(value)效果相同，转换成字符串格式化。
参考：[]!https://www.cnblogs.com/sesshoumaru/p/6005368.html
```
#整形数值可以提供的参数有 'b' 'c' 'd' 'o' 'x' 'X' 'n' None
>>> format(3,'b') #转换成二进制
'11'
>>> format(97,'c') #转换unicode成字符
'a'
>>> format(11,'d') #转换成10进制
'11'
>>> format(11,'o') #转换成8进制
'13'
>>> format(11,'x') #转换成16进制 小写字母表示
'b'
>>> format(11,'X') #转换成16进制 大写字母表示
'B'
>>> format(11,'n') #和d一样
'11'
>>> format(11) #默认和d一样
'11'
```

* 'frozenset',
传入一个可迭代对象，生成一个新的不可变集合。 
```
>>> a = frozenset(range(10))
>>> a
frozenset({0, 1, 2, 3, 4, 5, 6, 7, 8, 9})
```

* 'globals', 
所有的全局变量

* 'hasattr', 
函数功能用来检测对象object中是否含有名为name的属性，如果有则返回True，如果没有返回False

* 'hash', 
生成一个hash值（字符串）

* 'help',
快速获取模块,对象提供的功能

* 'hex',
将十进制转为十六进制  0x 表示十六进制

* 'id', 
 函数用于获取对象的内存地址

* 'input', 
如果提供了promat参数，首先将参数值输出到标准的输出，并且不换行。函数读取用户输入的值，将其转换成字符串。
```
>>> s = input('please input your name:')
please input your name:Ain
>>> s
'Ain'
```

* 'int', 
把其他类型转换为整数

* 'isinstance', 
isinstance(object, classinfo)
函数功能用于判断对象是否是类型对象的实例，object参数表示需要检查的对象，calssinfo参数表示类型对象。如果object参数是classinfo类型对象(或者classinfo类对象的直接、间接、虚拟子类)的实例，返回True。

* 'issubclass', 
issubclass(class, classinfo)
函数功能用于判断一个类型对象是否是另一个类型对象的子类，class参数表示需要检查的类型对象，calssinfo参数表示需要对比类型对象。如果class参数是classinfo类型对象(或者classinfo类对象的直接、间接、虚拟子类)的实例，返回True

* 'iter', 
 函数功能返回一个迭代器对象。当第二个参数不提供时，第一个参数必须是一个支持可迭代协议(即实现了__iter__()方法)的集合(字典、集合、不可变集合)，或者支持序列协议(即实现了__getitem__()方法，方法接收一个从0开始的整数参数)的序列(元组、列表、字符串)，否则将报错。
 ```
 l = [1, 2, 3]
  for i in iter(l):
      print(i)
 ```
如果传递了第二个参数，则object必须是一个可调用的对象（如，函数）。此时，iter创建了一个迭代器对象，每次调用这个迭代器对象的__next__()方法时，都会调用object。
如果__next__的返回值等于sentinel，则抛出StopIteration异常，否则返回下一个值
```
    class TestIter(object):
 
        def __init__(self):
            self.l=[1,2,3,4,5]
            self.i=iter(self.l)
        def __call__(self):  #定义了__call__方法的类的实例是可调用的
            item = next(self.i)
            print ("__call__ is called,which would return",item)
            return item
        def __iter__(self): #支持迭代协议(即定义有__iter__()函数)
            print ("__iter__ is called!!")
            return iter(self.l)
 
    t = TestIter()  # t是可调用的
    t1 = iter(t, 3)  # t必须是callable的，否则无法返回callable_iterator
    print(callable(t))
    for i in t1:
        print(i)
# 它每次在调用的时候，都会调用__call__函数，并且最后输出3就停止了。
```

* 'len', 
输出对象的长度

* 'license', 
打印license

* 'list', 
列表，可将元组，字符串等转换为列表。

* 'locals', 
所有的局部变量

* 'map', 
将函数返回值添加结果中,(函数，可迭代的对象（可以for循环）)
```
def f1(args):
    result = []
    for i in args:
        result.append(i + 100)
    return result
lst1 = [11,22,33,44,55,66]
rest = f1(lst1)
print (list(rest))
# [111, 122, 133, 144, 155, 166]
 
# 优化示例2，，,map函数
def f2(a):
    return a + 100
lst2 = [11,22,33,44,55,66]
result1 = map(f2,lst2)
print (list(result1))
# [111, 122, 133, 144, 155, 166]
 
# 优化示例2，map函数+lambda函数
lst3 = [11,22,33,44,55,66]
result2 = map(lambda a:a+100,lst3)
print (list(result2))
# [111, 122, 133, 144, 155, 166]
```

* 'max', 
最大值
```
lit = [11,22,33,44,55]
print (max(lit))
print (min(lit))
print (sum(lit))
```

* 'memoryview', 
该函数会创建一个引用自 obj 的内存视图(memoryview)对象。内存视图对象允许 Python 代码访问支持缓冲区协议(buffer protocol)的对象的内部数据，且无需拷贝。
obj 必须支持缓冲区协议(buffer protocol)。支持缓冲区协议的内置对象有 bytes 和 bytearray。bytes 和 bytearray 由 memoryview 提供支持，内存视图使用缓冲区协议(buffer protocol)访问来访问其它二进制对象的内存，且无需拷贝。
示例 - 使用内存视图对象修改一个短整型有符号整数数组的数据
```
from array import array

numbers = array('h', [-2, -1, 0, 1, 2]) # 'h'表示signed short
memv = memoryview(numbers) 
len(memv) #> 5
# 转换成列表形式
memv.tolist() #> [-2, -1, 0, 1, 2]
# 转换成无符号字符类型
memv_oct = memv.cast('B')       
memv_oct.tolist() #> [254, 255, 255, 255, 0, 0, 1, 0, 2, 0]
# 修改第3个元素的高位字段
memv_oct[5] = 4                 
numbers #> array('h', [-2, -1, 1024, 1, 2])
```

* 'min', 
最小值

* 'next', 
 函数必须接收一个可迭代对象参数，每次调用的时候，返回可迭代对象的下一个元素。如果所有元素均已经返回过，则抛出StopIteration 异常

* 'object', 
Object类是Python中所有类的基类，如果定义一个类时没有指定继承那个类，则默认继承object类

* 'oct', 
将十进制转为八进制  0o 表示八进制

* 'open',
open函数，该函数用于处理文件,参考[]!https://www.cnblogs.com/sesshoumaru/p/6047046.html

* 'ord',
见'chr'

* 'pow', 
求指数
```
print(pow(2,10))
```

* 'print', 
用于打印输出，最常见的一个函数
参考：[]!http://www.runoob.com/python/python-func-print.html

* 'property',
* 'setattr', 
* 'getattr', 
* 'delattr', 
property()函数中的三个函数分别对应的是获取属性的方法、设置属性的方法以及删除属性的方法，这样一来，外部的对象就可以通过访问x的方式，来达到获取、设置或删除属性的目的。
```
class Shuxing():
    def __init__(self, size = 10):
        self.size = size
    def getSize(self):
        return self.size
    def setSize(self, value):
        self.size = value
    def delSize(self):
        del self.size
    x = property(getSize, setSize, delSize)

>>> sx = Shuxing(100)
>>> sx.x
100
>>> sx.x= 106
>>> sx.x
106
>>> del sx.x
>>> sx.x
Traceback (most recent call last):
  File "<pyshell#60>", line 1, in <module>
    sx.x
  File "<pyshell#54>", line 5, in getSize
    return self.size
AttributeError: 'Shuxing' object has no attribute 'size'
```

* 'quit', 
同exit()

* 'range', 
可创建一个整数列表，一般用在 for 循环中
语法：range(start, stop[, step])
start: 计数从 start 开始。默认是从 0 开始。例如range（5）等价于range（0， 5）; 
stop: 计数到 stop 结束，但不包括 stop。例如：range（0， 5） 是[0, 1, 2, 3, 4]没有5 
step：步长，默认为1。例如：range（0， 5） 等价于 range(0, 5, 1)
```
>>>range(10)        # 从 0 开始到 10
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

* 'repr', 
函数功能返回一个对象的字符串表现形式。其功能和str函数比较类似，但是两者也有差异：函数str() 用于将值转化为适于人阅读的形式，而repr() 转化为供解释器读取的形式。
```
>>> a = 'some text'
>>> str(a)
'some text'
>>> repr(a)
"'some text'"
```

* 'reversed', 
反转
```
lit1 = [11,22,33,44,55,66]
print (list(reversed(lit1)))
# [66, 55, 44, 33, 22, 11]
```
'round', 
四舍五入求值

* 'set',
 函数创建一个无序不重复元素集，可进行关系测试，删除重复数据，还可以计算交集、差集、并集等 
 
* 'slice', 
slice() 函数实现切片对象，主要用在切片操作函数里的参数传递。

* 'sorted', 
排序 等同于列表的sort

* 'staticmethod', 
* 'str',
字节转化为字符串
```
# 将字符串转换为字节类型,系统中的表现形式为16进制
# bytes(字符串，编码格式)
s = "中国"
n3 = bytes(s,encoding="utf-8")
print (n3)
# b'\xe4\xb8\xad\xe5\x9b\xbd'
n4 = bytes(s,encoding="gbk")
print (n4)
# b'\xd6\xd0\xb9\xfa'

n5 = str(b'\xd6\xd0\xb9\xfa',encoding="gbk")
print (n5)
# 中国
n6 = str(b'\xe4\xb8\xad\xe5\x9b\xbd',encoding="utf-8")
print (n6)
# 中国
``` 
'sum', 
求和

* 'super', 
函数是用于调用父类(超类)的一个方法
Python3.x 和 Python2.x 的一个区别是: Python 3 可以使用直接使用 super().xxx 代替 super(Class, self).xxx

* 'tuple', 
将列表转换为元组

* 'type', 
type() 函数如果你只有第一个参数则返回对象的类型，三个参数返回新的类型对象。 
isinstance() 与 type() 区别：
type() 不会认为子类是一种父类类型，不考虑继承关系。
isinstance() 会认为子类是一种父类类型，考虑继承关系。 
如果要判断两个类型是否相同推荐使用 isinstance()。

* 'vars', 
返回对象object的属性和属性值的字典对象

* 'zip',
函数用于将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表
```
l1 = ["hello",11,22,33]
l2 = ["world",44,55,66]
l3 = ["python",77,88,99]
l4 = zip(l1,l2,l3)
# print (list(l4))
# # [('hello', 'world', 'python'), (11, 44, 77), (22, 55, 88), (33, 66, 99)]
temp1 = list(l4)[0]
print (temp1[0])
ret1 = " ".join(temp1)
print (ret1)
# hello world python
```