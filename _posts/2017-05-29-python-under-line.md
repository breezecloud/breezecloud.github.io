﻿---
layout: post
title:  "python的下划线使用"
date: 2017-05-29 16:25:06 -0700
---

&emsp;&emsp;最近在看一份python代码，对满屏的下划线相当困惑：单下划线、双下划线、双下划线还分前后......那它们的作用与使用场景 到底有何区别呢？。于是问了度娘，记录下来以便学习。并非完全原创，可以随意分享。

1. 单下划线"_"  
　　通常情况下，单下划线"_"会在以下3种场景中使用：
   *  在解释器中：  
　　在这种情况下，"_"代表交互式解释器会话中上一条执行的语句的结果。这种用法首先被标准CPython解释器采用，然后其他类型的解释器也先后采用。
   * 作为一个名称：  
　　这与上面一点稍微有些联系，此时"_"作为临时性的名称使用，但是并不会在后面再次用到该名称。例如，下面的例子中。
```
n = 42
for _ in range(n): 
　　do_something()　　
```
   * 国际化：  
　　也许你也曾看到"_"会被作为一个函数来使用。这种情况下，它通常用于实现国际化和本地化字符串之间翻译查找的函数名称，这似乎源自并遵循相应的C约定。例如，在 Django文档"转换"章节 中，你将能看到如下代码：
```
from django.utils.translation import ugettext as _ 
from django.http import HttpResponse 
def my_view(request): 
    output = _("Welcome to my site.") 
　　return HttpResponse(output)　
```
2. 名称前的单下划线（如：```_shahriar```）  
　　程序员使用名称前的单下划线，用于指定该名称属性为"私有"。这有点类似于惯例，为了使其他人（或你自己）使用这些代码时将会知道以"_"开头的名称只供内部使用。正如Python文档中所述：
　　以下划线"_"为前缀的名称（如_spam）应该被视为API中非公开的部分（不管是函数、方法还是数据成员）。此时，应该将它们看作是一种实现细节，在修改它们时无需对外部通知。  
　　如果你写了代码"from <模块/包名> import *"，那么以"_"开头的名称都不会被导入， 除非模块或包中的"__all__"列表显式地包含了它们 。了解更多请查看" Importing * in Python "。不过值得注意的是，如果使用 import a_module 这样导入模块，仍然可以用 a_module._some_var 这样的形式访问到这样的对象。  
　　另外单下划线开头还有一种一般不会用到的情况在于使用一个 C 编写的扩展库有时会用下划线开头命名，然后使用一个去掉下划线的 Python 模块进行包装。如 struct 这个模块实际上是 C 模块 _struct 的一个 Python 包装。  
3. 名称前的双下划线（如：```__shahriar```）  
　　名称（具体为一个方法名）前双下划线（__）的用法并不是一种惯例，对解释器来说它有特定的意义。Python中的这种用法是为了避免与子类定义的名称冲突。Python文档指出，"__spam"这种形式（ 至少两个前导下划线，最多一个后续下划线 ）的任何标识符将会被"_classname__spam"这种形式原文取代，在这里"classname"是去掉前导下划线的当前类名。例如下面的例子：
```
>>> class A(object): 
...     def _internal_use(self): 
...         pass
...     def __method_name(self): 
...         pass
... 
>>> dir(A()) 
['_A__method_name', ..., '_internal_use']
```
　　正如所预料的，"_internal_use"并未改变，而"__method_name"却被变成了"_ClassName__method_name"：__开头 的 私有变量会在代码生成之前被转换为长格式（变为公有）。转换机制是这样的：在变量前端插入类名，再在前端加入一个下划线字符。这就是所谓的私有变量 名字改编 （Private name mangling） 。
　　你可以在类内部使用原来定义的名字。此时，如果你创建A的一个子类B，那么你将不能轻易地覆写A中的方法"__method_name"，
```
>>> class B(A): 
...     def __method_name(self): 
...         pass
... 
>>> dir(B()) 
['_A__method_name', '_B__method_name', ..., '_internal_use']
```
　　然而如果你知道了这个规律，最终你还是可以访问这个"私有"变量的。要注意的是混淆规则（私有变量名字改编）主要目的在于避免意外错误，被认作为私有的变量仍然有可能被访问或修改(使用_classname__membername)，在特定的场合它也是有用的，比如调试的时候。  
　　另外：  
　　一、是因为轧压（改编）会使标识符变长，当超过255的时候，Python会切断，要注意因此引起的命名冲突。  
　　二、是当类名全部以下划线命名的时候，Python就不再执行轧压（改编）。  
　　无论是单下划线还是双下划线开头的成员，都是希望外部程序开发者不要直接使用这些成员变量和这些成员函数，只是双下划线从语法上能够更直接的避 免错误的使用，但是如果按照 _类名__成员名 则依然可以访问到。单下划线的在动态调试时可能会方便一些，只要项目组的人都遵守下划线开头的成员不直接使用，那使用单下划线或许会更好。
4. 名称前后的双下划线（如：```__init__```）  
　　这种用法表示Python中特殊的方法名。其实，这只是一种惯例，对Python系统来说，这将确保不会与用户自定义的名称冲突。通常，你将会 覆写这些方法，并在里面实现你所需要的功能，以便Python调用它们。例如，当定义一个类时，你经常会覆写"__init__"方法。
双下划线开头双下划线结尾的是一些 Python 的"魔术"对象，如类成员的 ```__init__、__del__、__add__、__getitem__ ```等，以及全局的 ```__file__、__name__ ```等。 Python 官方推荐永远不要将这样的命名方式应用于自己的变量或函数，而是按照文档说明来使用。 虽然你也可以编写自己的特殊方法名，但不要这样做。
5. 题外话 ```if __name__ == "__main__": ``` 
　　所有的 Python 模块都是对象并且有几个有用的属性，你可以使用这些属性方便地测试你所书写的模块。
模块是对象, 并且所有的模块都有一个内置属性 __name__。一个模块的 ```__name__``` 的值要看您如何应用模块。如果 import 模块, 那么 ```__name__```的值通常为模块的文件名, 不带路径或者文件扩展名。但是您也可以像一个标准的程序一样直接运行模块, 在这种情况下 __name__的值将是一个特别的缺省值：```__main__```。
```
>>> import odbchelper
>>> odbchelper.__name__
'odbchelper'
```
　　一旦了解到这一点, 您可以在模块内部为您的模块设计一个测试套件, 在其中加入这个 if 语句。当您直接运行模块, ```__name__``` 的值是 ```__main__```, 所以测试套件执行。当您导入模块, __name__的值就是别的东西了, 所以测试套件被忽略。这样使得在将新的模块集成到一个大程序之前开发和调试容易多了。  
　　在 MacPython 上, 需要一个额外的步聚来使得 if __name__ 技巧有效。 点击窗口右上角的黑色三角, 弹出模块的属性菜单, 确认 Run as __main__ 被选中。
6. 总结： 
``` 
_xxx       不能用'from module import *'导入  
__xxx__  系统定义名字  
__xxx     类中的私有变量名 
``` 
　　因为下划线对解释器有特殊的意义，而且是内建标识符所使用的符号，我们建议程序员避免用下划线作为变量名的开头。一般来讲，变量名_xxx被看作是"私有的"，在模块或类外不可以使用。当变量是私有的时候，用_xxx 来表示变量是很好的习惯。 因为变量名__xxx__对Python 来说有特殊含义，对于普通的变量应当避免这种命名风格。
　　"单下划线" 开始的成员变量叫做保护变量，意思是只有类对象和子类对象自己能访问到这些变量；  
　　"双下划线" 开始的是私有成员，意思是只有类对象自己能访问，连子类对象也不能访问到这个数据。  
　
&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin.jpg)
