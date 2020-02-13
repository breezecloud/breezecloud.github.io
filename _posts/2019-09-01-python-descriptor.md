---
layout: post
title:  "python属性查找（attribute lookup）是怎么回事？"
date: 2019-09-01 08:15:00 +0800
categories: jekyll update
---

python属性查找（attribute lookup）是怎么回事？
===
学习python，绕不过学习python的属性查找问题。原来引用python的属性不是简单的该对象的属性值，它有一定的规则来判断到底取那个对象的属性或方法的值，涉及到描述符（descriptor）和装饰器（Decorators）方面的知识。但具体什么规则，查了好多网上文章，虽然说法不错，但因为写的角度不同，总觉得没说清楚。经过实验，终于搞清楚了背后的机制。<br>
以下内容转至：https://www.cnblogs.com/Jimmy1988/p/6808237.html<br>
在Python中，属性查找（attribute lookup）是比较复杂的，特别是涉及到描述符descriptor的时候。首先，我们知道：python中一切都是对象，“everything is object”，包括类，类的实例，数字，模块任何object都是类（class or type）的实例（instance）如果一个descriptor只实现了__get__方法，我们称之为non-data descriptor， 如果同时实现了__get__ __set__我们称之为data descriptor。按照python doc，如果obj是某个类的实例，那么obj.name首先调用__getattribute__。如果类定义了__getattr__方法，那么在__getattribute__抛出 AttributeError 的时候就会调用到__getattr__，而对于描述符(__get__）的调用，则是发生在__getattribute__内部的。<br>
obj = Clz(), 那么obj.attr 顺序如下：
```
        （1）如果“attr”是出现在Clz或其基类的__dict__中， 且attr是data descriptor， 那么调用其__get__方法, 否则
        （2）如果“attr”出现在obj的__dict__中， 那么直接返回 obj.__dict__['attr']， 否则
        （3）如果“attr”出现在Clz或其基类的__dict__中
            （3.1）如果attr是non-data descriptor，那么调用其__get__方法， 否则
            （3.2）返回 __dict__['attr']
        （4）如果Clz有__getattr__方法，调用__getattr__方法，否则
        （5）抛出AttributeError 
```        
    　　下面是测试代码：
```    
    #coding=utf-8
    class DataDescriptor(object):
        def __init__(self, init_value):
            self.value = init_value
      
        def __get__(self, instance, typ):
            return 'DataDescriptor __get__'
      
        def __set__(self, instance, value):
            print ('DataDescriptor __set__')
            self.value = value
     
    class NonDataDescriptor(object):
        def __init__(self, init_value):
            self.value = init_value

        def __get__(self, instance, typ):
            return('NonDataDescriptor __get__')
     
    class Base(object):
        dd_base = DataDescriptor(0)
        ndd_base = NonDataDescriptor(0)

     
    class Derive(Base):
        dd_derive = DataDescriptor(0)
        ndd_derive = NonDataDescriptor(0)
        same_name_attr = 'attr in class'
     
        def __init__(self):
            self.not_des_attr = 'I am not descriptor attr'
            self.same_name_attr = 'attr in object'
     
        def __getattr__(self, key):
            return '__getattr__ with key %s' % key
     
        def change_attr(self):
            self.__dict__['dd_base'] = 'dd_base now in object dict '
            self.__dict__['ndd_derive'] = 'ndd_derive now in object dict '
     
    def main():
        b = Base()
        d = Derive()
        print('Derive object dict', d.__dict__)
        assert d.dd_base == "DataDescriptor __get__"
        assert d.ndd_derive == 'NonDataDescriptor __get__'
        assert d.not_des_attr == 'I am not descriptor attr'
        assert d.no_exists_key == '__getattr__ with key no_exists_key'
        assert d.same_name_attr == 'attr in object'
        d.change_attr()
        print('Derive object dict', d.__dict__)
        assert d.dd_base != 'dd_base now in object dict '
        assert d.ndd_derive == 'ndd_derive now in object dict '

        try:
            b.no_exists_key
        except Exception, e:
            assert isinstance(e, AttributeError)
     
    if __name__ == '__main__':
        main()
```
        调用change_attr方法之后，dd_base既出现在类的__dict__（作为data descriptor）, 也出现在实例的__dict__， 因为attribute lookup的循序，所以优先返回的还是Clz.__dict__['dd_base']。而ndd_base虽然出现在类的__dict__， 但是因为是nondata descriptor，所以优先返回obj.__dict__['dd_base']。其他：line48,line56表明了__getattr__的作用。line49表明obj.__dict__优先于Clz.__dict__
        前面提到过，类也是对象，类是元类（metaclass）的实例，所以类属性的查找顺序基本同上，区别在于第二步，由于Clz可能有基类，所以是在Clz及其基类的__dict__查找“attr"<br>
        补充说明：很多网上说属性最高优先级是__getattribute__。但一般我们不会去覆盖__getattribute__，因为一旦覆盖了__getattribute__，所有后面的规则都失效了，所有要理解上面提到的"生在__getattribute__内部"。<br>
       再看以下代码：
```       
    import functools, time
    class cached_property(object):
        """ A property that is only computed once per instance and then replaces
            itself with an ordinary attribute. Deleting the attribute resets the
            property. """

        def __init__(self, func):
            functools.update_wrapper(self, func)
            self.func = func

        def __get__(self, obj, cls):
            if obj is None: return self
            value = obj.__dict__[self.func.__name__] = self.func(obj)
            return value

    class TestClz(object):
        @cached_property
        def complex_calc(self):
            print('very complex_calc')
            return sum(range(100))

    if __name__=='__main__':
        t = TestClz()
        print('>>> first call')
        print(t.complex_calc)
        print('>>> second call')
        print(t.complex_calc)
```
    cached_property是一个non-data descriptor。在TestClz中，用cached_property装饰方法complex_calc，返回值是一个descriptor实例，所以在调用的时候没有使用小括号。第一次调用t.complex_calc之前，obj(t)的__dict__中没有”complex_calc“， 根据查找顺序第三条，执行cached_property.__get__, 这个函数代用缓存的complex_calc函数计算出结果，并且把结果放入obj.__dict__。那么第二次访问t.complex_calc的时候，根据查找顺序，第二条有限于第三条，所以就直接返回obj.__dict__['complex_calc']。<br>
再举一个网上常见的例子：
```
    class NotNegative():
        def __init__(self,name):
            self.name = name

        def __set__(self, instance, value):
            if value < 0:
                raise ValueError(self.name+' must be >= 0')
            else:
                instance.__dict__[self.name] = value

    class Product():
        quantity = NotNegative('quantity')
        price = NotNegative('price')

        def __init__(self,name,quantity,price):
            self.name = name
            self.quantity = quantity
            self.price = price

    book = Product('mybook',2,5)
```
        通过描述符来限制设置quantity和price必须是非负。在该例子中，如果执行book.quantity=3,解释器会先查找实例属性，发现有quantity属性，但是解释器又发现同样有一个类属性是描述符，于是解释器最终会选择走描述符这条路。然后因为是描述符，于是会执行描述符中的set特殊方法。<br>

        描述符中的set特殊方法的参数有为<br>
```        
        self ：是描述符实例
        instance ：是相当于例子中的实例book
        value ：就是要赋予的值
```
get方法同样有3个参数self, instance, owner。self，instance与set中的相同，owner为例子中的Product类<br>

python的描述符和装饰器功能强大，用好了极大地增强了代码的复用性和可读性，但同时也是比较难学的知识点，多看看别人的代码可以帮助我们更快的理解。