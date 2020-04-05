---
title: Python装饰器
description: 记录Python中装饰器的使用，以及不同写法的区别。
date: 2020-04-05 10:08:19
categories:
  - Python
tags:
  - 装饰器
---
# 理解

装饰器，顾名思义就是用来包装的，在表面进行修饰的方式。

装饰器本身就是一种函数或类，返回值同样也是函数或类。它的意义主要在于无需对原有代码进行修改的前提下添加一些额外的功能，采用一种切面编程思想，比如常用的应用场景日志记录、性能测试、事物处理、权限校验、缓存等功能。有了装饰器我们能够尽可能的重用代码，避免大量重复代码，有效的规避规避对现有业务代码进行改动导致新的产生。

# Demo

本文主要演示手动实现和采用Python语法糖实现方式。

## 手动实现

1.简单函数调用

``` Python
def test(func):
  print('begin execution func.')
  func()
  print('func execution ends.')
  
def hello():
  print('hello.')
  
test(hello)
```

打印输出：

> begin execution func.
>
> hello.
>
> func execution ends.

2.日志记录

``` python
import time

def logger(func):
  def wrapper():
    print('[%d]start run %s.' % (int(time.time()), func.__name__))
    return func()
  return wrapper

def test():
  print('wait 10 seconds.')
  time.sleep(10)
  
hello = logger(test)
hello()
```

打印输出：

> [1563619707]start run test.
>
> wait 10 seconds.

上面例子中，`logger`函数中的`wrapper`函数将原`test`函数包装后返回，最后调用执行。

## @ 语法糖

如果你了解Python语法，一定看到或使用过`@`符号，`@`就是Python中装饰器的语法糖。使用时反正目标函数定义的地方，这样即可省略掉赋值的过程。

1.无参装饰器

``` python
import time

def logger(func):
  def wrapper():
    print('[%d]start run %s.' % (int(time.time()), func.__name__))
    return func()
  return wrapper

@logger
def test():
  print('wait 10 seconds.')
  time.sleep(10)
  
test()
```

打印输出：

> [1563621964]start run test.
>
> wait 10 seconds.

以上例子采用装饰器的方式可以无侵入性的添加或实现额外的功能。

2.带参目标函数装饰器写法

- 单个参数

  ``` python
  def logger(func):

    def wrapper(name):
        print('start run %s' % func.__name__)
        return func(name)
    return wrapper

  @logger
  def test(name):
      print('my name %s' % name)

  test('this blog')
  ```

- 多个参数

  ``` python
  def logger(func):
    
  	def wrapper(*args):
    		print('start run %s' % func.__name__)
      	return func(*args)
    return wrapper
  
  @logger
  def test(name, age):
    	print('my name %s,age %d' % (name, age))
    
  test('this blog', 1)
  ```

  

- 关键字参数

  ``` python
  def logger(func):
    
  	def wrapper(*args, **kwargs):
    		print('start run %s' % func.__name__)
      	return func(*args, **kwargs)
    return wrapper
  
  @logger
  def test(name, age, phone=None, address=None):
    	print('my name %s,age %d,phone %d,address %s' % (name, age, phone, address))
    
  test('this blog', 1, phone=124334, address="test address")
  ```

- 带参装饰器

  ``` python
  def logger(level):
    def decorator(func):
      
      def wrapper(*args, **kwargs):
        print('[%s]start run %s' % (level, func.__name__))
      	return func(*args, **kwargs)
    	return wrapper
    return decorator
  
  @logger(level="INFO")
  def test(name, age, phone=None, address=None):
    	print('my name %s,age %d,phone %d,address %s' % (name, age, phone, address))
    
  test('this blog', 1, phone=124334, address="test address")
  ```

  打印信息：

  > [INFO]start run test
  >
  > my name this blog,age 1,phone 124334,address test address

  带参装饰器返回的本就是一个装饰器，`@logger(level="INFO")`等同于`decorator`


 3.类装饰器

装饰器不仅仅可以是函数，也可是类。类装饰器具有更大的灵活性、高类聚、封装性等优点。类装饰器主要利用类的一个`__call__`方法实现，`@`形式装饰器附加到函数上时就会调用该方法。

  *一个对象为可调用对象`callable`时就会实现`__call__`方法* 

  ``` python
  import time
  
  class logger:
    
    def __init__(self, func):
      	self.func = func
    
    def __call__(self, *args, **kwargs):
      	print('[args]name %s, age %d' % *args)
      	print('start run %s:%d' % (self.func.__name__, int(time.time())))
      	self.func(*args, **kwargs)
      	print('%s running is complete:%d' % (self.func.__name__, int(time.time())))
            
  @logger
  def test(name, age):
    	print('my is name %s, age %d' % (name, age))
      print('wait 10 seconds.')
      time.sleep(10)
  
  test('this blog', 1)
  ```

  打印输出：

  > [args]ame this blog, age 1
  >
  > start run test:1563626058
  >
  > my is name this blog, age 1
  >
  > wait 10 seconds.
  >
  > test running is complete:1563626068

  在执行带参目标函数时，参数会传递给类装饰器的`__call__`方法，然后我们将参数传递给目标方法执行。

4.多个装饰器使用

一个函数可以同时使用多个装饰器，执行顺序为由内而外。

``` python
@func3
@func2
@func1
def test():
  pass

# 等效使用方式
func = func3(func2(func1(test)))
```