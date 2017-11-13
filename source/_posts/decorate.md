---
title: python装饰器学习
date: 2017-11-13 14:59:19
tags: python
categories: python学习
---

## 1 装饰器初试

### 1.1 装饰器基础知识

装饰器是可调用的对象，其参数是另一个函数（被装饰的函数）。装饰器可能会处理被装饰的函数，然后把它返回，或者将其替换成另一个函数或可调用对象。
<!--more-->

```python
@decorate
def target():
    print('running target()')
```

上述代码和下述代码的写法一样


```python
def target():
    print('running target()')
target = decorate(target)
```

两种写法的最终结果一样：上述两个代码片段执行完毕后得到的target不一定是原来那个target 函数，而是decorate(target)返回的函数。


```python
def deco(func):
    def inner():
        print('running inner()')
    return inner
@deco
def target():
    print('running target()')
```


```python
target()
print(target)
```

    running inner()
    <function deco.<locals>.inner at 0x0000016C2B20AA60>


严格来说，装饰器只是语法糖。如前所示，装饰器可以像常规的可调用对象那样调用，其参数是另一个函数。有时，这样做更方便，尤其是做元编程（在运行时改变程序的行为）时。综上，装饰器的一大特性是，能把被装饰的函数替换成其他函数。第二个特性是，装饰器在加载模块时立即执行。下一节会说明。

### 1.2 Python何时执行装饰器

装饰器的一个关键特性是， 它们在被装饰的函数定义之后立即运行。 这通常是在导入时(即 Python 加载模块时） ， 如 registration.py 模块所示。

```python
# BEGIN REGISTRATION

registry = []  # <1>

def register(func):  # <2>
    print('running register(%s)' % func)  # <3>
    registry.append(func)  # <4>
    return func  # <5>

@register  # <6>
def f1():
    print('running f1()')

@register
def f2():
    print('running f2()')

def f3():  # <7>
    print('running f3()')

def main():  # <8>
    print('running main()')
    print('registry ->', registry)
    f1()
    f2()
    f3()

if __name__=='__main__':
    main()  # <9>

# END REGISTRATION
```

    running register(<function f1 at 0x0000016C2B20A378>)
    running register(<function f2 at 0x0000016C2B23B048>)
    running main()
    registry -> [<function f1 at 0x0000016C2B20A378>, <function f2 at 0x0000016C2B23B048>]
    running f1()
    running f2()
    running f3()


注意， register 在模块中其他函数之前运行（两次） 。 调用 register 时， 传给它的参数是被装饰的函数， 例如 <function f1 at 0x100631bf8>。
加载模块后， registry 中有两个被装饰函数的引用： f1 和 f2。 这两个函数， 以及 f3，只在 main 明确调用它们时才执行。如果导入 registration.py 模块（不作为脚本运行） ， 输出如下：


```python
>>> import registration
running register(<function f1 at 0x10063b1e0>)
running register(<function f2 at 0x10063b268>)
```

此时查看 registry 的值， 得到的输出如下


```python
>>> registration.registry
[<function f1 at 0x10063b1e0>, <function f2 at 0x10063b268>]
```

主要想强调， 函数装饰器在导入模块时立即执行， 而被装饰的函数只在明确调用时运行。 这突出了 Python 程序员所说的导入时和运行时之间的区别。

### 1.3 装饰器改进“策略”模式

定义体中有函数的名称， 但是 best_promo 用来判断哪个折扣幅度最大的 promos 列表中也有函数名称。 这种重复是个问题， 因为新增策略函数后可能会忘记把它添加到 promos 列表中， 导致 best_promo 忽略新策略， 而且不报错， 为系统引入了不易察觉的缺陷。 

`实例 1` promos 列表中使用promotion装饰器填充


```python
# strategy_best4.py
# Strategy pattern -- function-based implementation
# selecting best promotion from list of functions
# registered by a decorator

"""
    >>> joe = Customer('John Doe', 0)
    >>> ann = Customer('Ann Smith', 1100)
    >>> cart = [LineItem('banana', 4, .5),
    ...         LineItem('apple', 10, 1.5),
    ...         LineItem('watermellon', 5, 5.0)]
    >>> Order(joe, cart, fidelity)
    <Order total: 42.00 due: 42.00>
    >>> Order(ann, cart, fidelity)
    <Order total: 42.00 due: 39.90>
    >>> banana_cart = [LineItem('banana', 30, .5),
    ...                LineItem('apple', 10, 1.5)]
    >>> Order(joe, banana_cart, bulk_item)
    <Order total: 30.00 due: 28.50>
    >>> long_order = [LineItem(str(item_code), 1, 1.0)
    ...               for item_code in range(10)]
    >>> Order(joe, long_order, large_order)
    <Order total: 10.00 due: 9.30>
    >>> Order(joe, cart, large_order)
    <Order total: 42.00 due: 42.00>

# BEGIN STRATEGY_BEST_TESTS

    >>> Order(joe, long_order, best_promo)
    <Order total: 10.00 due: 9.30>
    >>> Order(joe, banana_cart, best_promo)
    <Order total: 30.00 due: 28.50>
    >>> Order(ann, cart, best_promo)
    <Order total: 42.00 due: 39.90>

# END STRATEGY_BEST_TESTS
"""

from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity')


class LineItem:

    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.price * self.quantity


class Order:  # the Context

    def __init__(self, customer, cart, promotion=None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion

    def total(self):
        if not hasattr(self, '__total'):
            self.__total = sum(item.total() for item in self.cart)
        return self.__total

    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion(self)
        return self.total() - discount

    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(), self.due())

# BEGIN STRATEGY_BEST4

promos = []  # <1>

def promotion(promo_func):  # <2>
    promos.append(promo_func)
    return promo_func

@promotion  # <3>
def fidelity(order):
    """5% discount for customers with 1000 or more fidelity points"""
    return order.total() * .05 if order.customer.fidelity >= 1000 else 0

@promotion
def bulk_item(order):
    """10% discount for each LineItem with 20 or more units"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

@promotion
def large_order(order):
    """7% discount for orders with 10 or more distinct items"""
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * .07
    return 0

def best_promo(order):  # <4>
    """Select best discount available
    """
    return max(promo(order) for promo in promos)

# END STRATEGY_BEST4
```

`优点如下：`

- 促销策略函数无需使用特殊的名称（即不用以 _promo 结尾）。
- @promotion 装饰器突出了被装饰的函数的作用， 还便于临时禁用某个促销策略： 只需把装饰器注释掉。
- 促销折扣策略可以在其他模块中定义， 在系统中的任何地方都行， 只要使用@promotion 装饰即可。

## 2 闭包

### 2.1 闭包的使用

在博客圈， 人们有时会把闭包和匿名函数弄混。 这是有历史原因的： 在函数内部定义函数不常见， 直到开始使用匿名函数才会这样做。 而且， 只有涉及嵌套函数时才有闭包问题。因此， 很多人是同时知道这两个概念的。其实， 闭包指延伸了作用域的函数， 其中包含函数定义体中引用、 但是不在定义体中定义的非全局变量。 函数是不是匿名的没有关系， 关键是它能访问定义体之外定义的非全局变量。

`1、闭包实现`


```python
"""
>>> avg = make_averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
>>> avg.__code__.co_varnames
('new_value', 'total')
>>> avg.__code__.co_freevars
('series',)
>>> avg.__closure__  # doctest: +ELLIPSIS
(<cell at 0x...: list object at 0x...>,)
>>> avg.__closure__[0].cell_contents
[10, 11, 12]
"""

DEMO = """
>>> avg.__closure__
(<cell at 0x107a44f78: list object at 0x107a91a48>,)
"""


def make_averager():
    series = []

    def averager(new_value):
        series.append(new_value)
        total = sum(series)
        return total/len(series)

    return averager

```

`2、初学者实现`


```python
"""
>>> avg = Averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0

"""


class Averager():

    def __init__(self):
        self.series = []

    def __call__(self, new_value):
        self.series.append(new_value)
        total = sum(self.series)
        return total/len(self.series)

```

综上， 闭包是一种函数， 它会保留定义函数时存在的自由变量的绑定， 这样调用函数时，虽然定义作用域不可用了， 但是仍能使用那些绑定。注意， 只有嵌套在其他函数中的函数才可能需要处理不在全局作用域中的外部变量。

### 2.2 nonloacl声明

前面实现 make_averager 函数的方法效率不高。在示例中，我们把所有值存储在历史数列中，然后在每次调用averager时使用sum求和。更好的实现方式是，只存储目前的总值和元素个数，然后使用这两个数计算均值。


```python
def make_averager():
    count = 0
    total = 0
    def averager(new_value):
        count += 1
        total += new_value
        return total / count
    return averager
```

尝试使用示例定义的函数， 会得到如下结果：


```python
>>> avg = make_averager()
>>> avg(10)
Traceback (most recent call last):
...
UnboundLocalError: local variable 'count' referenced before assignment
>>>
```

问题是， 当 count 是数字或任何不可变类型时， count += 1 语句的作用其实与 count= count + 1 一样。 因此， 我们在 averager 的定义体中为 count 赋值了， 这会把count 变成局部变量。 total 变量也受这个问题影响。

前面没有遇到这个问题， 因为我们没有给 series 赋值， 我们只是调用series.append， 并把它传给 sum 和 len。 也就是说， 我们利用了列表是可变的对象这一事实。但是对数字、 字符串、 元组等不可变类型来说， 只能读取， 不能更新。 如果尝试重新绑定， 例如 count = count + 1， 其实会隐式创建局部变量 count。 这样， count 就不是自由变量了， 因此不会保存在闭包中。

为了解决这个问题， Python 3 引入了 nonlocal 声明。 它的作用是把变量标记为自由变量， 即使在函数中为变量赋予新值了， 也会变成自由变量。 如果为 nonlocal 声明的变量赋予新值， 闭包中保存的绑定会更新。 最新版 make_averager 的正确实现如示例所示。


```python
def make_averager():
    count = 0
    total = 0
    def averager(new_value):
        nonlocal count, total
        count += 1
        total += new_value
        return total / count
    return averager
```

## 3 实现一个简单的装饰器

 定义了一个装饰器， 它会在每次调用被装饰的函数时计时， 然后把经过的时间、 传入的参数和调用的结果打印出来。


```python
# clockdeco.py
import time
def clock(func):
    def clocked(*args):
        t0 = time.time()
        result = func(*args)
        elapsed = time.time()-t0 #闭包中包含自由变量func()
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.8fs] %s(%s) -> $r' % \
              (elapsed, name, arg_str, result))
        return result
    return clocked #返回内部函数，取代被装饰的函数
```

使用clock装饰器


```python
# clockdeco_demo.py
import time
import clockdeco import clock

@clock
def snooze(seconds):
    time.sleep(seconds)

@clock
def factorial(n):
    return 1 if n < 2 else n*factorial(n-1)

if __name__ == '__main__':
    print('*' * 40, 'Calling snozzen(.123)')
    snooze(.123)
    print('*' * 40, 'Calling factorial(6)')
    factorial(6)
```


```python
$ python3 clockdeco_demo.py
**************************************** Calling snooze(123)
[0.12405610s] snooze(.123) -> None
**************************************** Calling factorial(6)
[0.00000191s] factorial(1) -> 1
[0.00004911s] factorial(2) -> 2
[0.00008488s] factorial(3) -> 6
[0.00013208s] factorial(4) -> 24
[0.00019193s] factorial(5) -> 120
[0.00026107s] factorial(6) -> 720
6! = 720
```

这是装饰器的典型行为： 把被装饰的函数替换成新函数， 二者接受相同的参数， 而且（通常） 返回被装饰的函数本该返回的值， 同时还会做些额外操作。

示例中实现的 clock 装饰器有几个缺点：不支持关键字参数，而且遮盖了被装饰函数的 \__name\__ 和 \__doc\__ 属性。下面使用 functools.wraps 装饰器把相关的属性从 func 复制到 clocked 中。此外，这个新版还能正确处理关键字参数。


```python
# clockdeco2.py
import time
import functools
def clock(func):
    @functions.wraps(func)
    def clocked(*args, **kwargs):
        t0 = time.time()
        result = func(*args)
        elapsed = time.time()-t0 #闭包中包含自由变量func()
        name = func.__name__
        arg_lst = []
        if args:
            arg_lst.append(', '.join(repr(arg) for arg in args))
        if kwargs:
            pairs = ['%s=%r' % (k, w) for k, w in sorted(kwargs.items())]
            arg_lst.append(', '.join(pairs))
        arg_str = ', '.join(arg_lst)
        print('[%0.8fs] %s(%s) -> %r ' % (elapsed, name, arg_str, result))
    return result
```

## 4 参数化装饰器

解析源码中的装饰器时， Python 把被装饰的函数作为第一个参数传给装饰器函数。 那怎么让装饰器接受其他参数呢？ 答案是： 创建一个装饰器工厂函数， 把参数传给它， 返回一个装饰器， 然后再把它应用到要装饰的函数上。 

### 4.1 一个参数化的注册装饰器

为了便于启用或禁用 register 执行的函数注册功能， 我们为它提供一个可选active参数， 设为 False 时， 不注册被装饰的函数。 实现方式参见示例。从概念上看， 这个新的 register 函数不是装饰器， 而是装饰器工厂函数。 调用它会返回真正的装饰器，这才是应用到目标函数上的装饰器。


```python
# BEGIN REGISTRATION_PARAM
registry = set()  # <1>

def register(active=True):  # <2>
    def decorate(func):  # <3>
        print('running register(active=%s)->decorate(%s)'
              % (active, func))
        if active:   # <4>
            registry.add(func)
        else:
            registry.discard(func)  # <5>

        return func  # <6>
    return decorate  # <7>

@register(active=False)  # <8>
def f1():
    print('running f1()')

@register()  # <9>
def f2():
    print('running f2()')

def f3():
    print('running f3()')

print(registry)
# END REGISTRATION_PARAM
```

代码在 registration_param.py 模块中。 如果导入， 得到的结果如下：


```python
>>> import registration_param
running register(active=False)->decorate(<function f1 at 0x10063c1e0>)
running register(active=True)->decorate(<function f2 at 0x10063c268>)
>>> registration_param.registry
{<function f2 at 0x10063c268>}
```

### 4.2 参数化clock装饰器

探讨 clock 装饰器， 为它添加一个功能： 让用户传入一个格式字符串， 控制被装饰函数的输出。


```python
# clockdeco_param.py

"""
>>> snooze(.1)  # doctest: +ELLIPSIS
[0.101...s] snooze(0.1) -> None
>>> clock('{name}: {elapsed}')(time.sleep)(.2)  # doctest: +ELLIPSIS
sleep: 0.20...
>>> clock('{name}({args}) dt={elapsed:0.3f}s')(time.sleep)(.2)
sleep(0.2) dt=0.201s
"""

# BEGIN CLOCKDECO_PARAM
import time

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

def clock(fmt=DEFAULT_FMT):  # <1>
    def decorate(func):      # <2>
        def clocked(*_args): # <3>
            t0 = time.time()
            _result = func(*_args)  # <4>
            elapsed = time.time() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)  # <5>
            result = repr(_result)  # <6>
            print(fmt.format(**locals()))  # <7>
            return _result  # <8>
        return clocked  # <9>
    return decorate  # <10>

if __name__ == '__main__':

    @clock()  # <11>
    def snooze(seconds):
        time.sleep(seconds)

    for i in range(3):
        snooze(.123)

# END CLOCKDECO_PARAM

```


```python
$ python3 clockdeco_param.py
[0.12412500s] snooze(0.123) -> None
[0.12411904s] snooze(0.123) -> None
[0.12410498s] snooze(0.123) -> None
```

使用了 clockdeco_param 模块中的新功能， 随后是两个模块输出的结果。


```python
import time
from clockdeco_param import clock

@clock('{name}: {elapsed}s')
def snooze(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)
```


```python
$ python3 clockdeco_param_demo1.py
snooze: 0.12414693832397461s
snooze: 0.1241159439086914s
snooze: 0.12412118911743164s
```