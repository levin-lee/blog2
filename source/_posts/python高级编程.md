---
title: python高级编程
date: 2020-08-21 19:32:32
tags:
  - python
  - 编程
---
## 1、filter函数-过滤
可以查看下面的例子
```python
num = [1,4,-5,3,-7,6]
after_filter = map(lambda x:x**2,filter(lambda x:x>0,num))
print(after_filter)
```
当然，也可以用python自带的迭代器,可以达到一样的效果
```python
after_filter = [x**2 for x in num if x>0]
```

此外，还可以使用生成器
```python
def squre_generator(parameter):
    rerurn(x**2 for x in num if x>parameter)
g = list(squre_generator(0))
```

## 2、装饰器
装饰器为我们提供了一个增加已有函数或类的功能。
```python
import time
from functools import wraps

def timethis(func):

   def wrapper(*args,**kwargs):
      start = time.time()
      result = func(*args,**kwargs)
      end = time.time()
      print(func.__name__,end-start)
      return wrapper

def countdown(n):
    while n>0:
    n-=1

countdown(10000)
```

## 3、_和和xx__的区别
### 1）“_”单下划线
可以在类的方法或属性前加一个“_”单下划线，意味着该方法或属性不应该去调用。
```python
class A: 
    def _method(self): 
        print('约定为不在类的外面直接调用这个方法，但是也可以调用’) 
    def method(self): 
        return self._method() 
a = A()
```
在类A中定义了一个_method方法，按照约定是不能在类外面直接调用它的，为了可以在外面使用_method方法，又定义了method方法，method方法调用_method方法。请看代码演示：
a._method() 不建议在类的外面直接调用这个方法，但是也可以调用。最好是a.method()

### 2）双”__”下划线
```python
class A:
    def __method(self):
        print('This is a method from class A')
    def method(self):
        return self.__method()
class B(A):
    def __method(self):
        print('This is a method from calss B')
```
在类A中，method方法其实由于name mangling技术的原因，变成了_Amethod，所以在A中method方法返回的是_Amethod，B作为A的子类，只重写了method方法，并没有重写method方法，所以调用B中的method方法时，调用的还是_Amethod方法：
```python
In [27]: a = A()
In [28]: b = B()
In [29]: a.method()
This is a method from class A
In [30]: b.method()
This is a method from class A
```
在A中没有method方法，有的只是_A__method方法，也可以在外面直接调用，所以python中没有真正的私有化

### 3）“xx”前后各双下划线
在特殊的情况下，它只是python调用的hook。例如，init()函数是当对象被创建初始化时调用的;new()是用来创建实例。
init #构造初始化函数,new之后运行
new #创建实例所需的属性
class #实例所在的类，实例.class
str #实例的字符串表示，可读性高
repr #实例的字符串表示，准确性高
del #删除实例引用
dict #实力自定义属性，vars(实例.dict)
doc #类文档，help(类或者实例)
bases #当前类的所有父类
getattribute #属性访问拦截器。

## 4、map函数
map()函数接收两个参数，一个是函数，一个是序列，map将传入的函数依次作用到序列的每个元素，并把结果作为新的list返回。
```python
def f(x):
return x * x
map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
```

## 5、reduce函数
reduce这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累积计算，其效果就是：
```python
def fn(x, y):
return x * 10 + y
reduce(fn, [1, 3, 5, 7, 9])
13579
```
