Filter
------

> filter(function, iterable)

__filter__ 函数接收一个函数和一个 可迭代对象, 并返回可迭代对象 _iterable_ 中的每一个元素在 _function_ 函数中运算返回 _True_ 的元素组成的新的迭代器. _iterable_ 可以是队列, 或者支持迭代的容器, 或者是一个迭代器. 如果 _function_ 函数为空( None ), 那么将 _function_ 设定为假定的函数, 这样, 对 _iterable_ 可迭代对象中的元素都会被认为返回 _False_ , 会在返回的迭代器中移除. 

```python
iter = [1, 2, 3, 4, 5, 6, 7, 8]
list(filter(lambda i: i%2, iter))
# Output: [1, 3, 5, 7]
```

<!--more-->

__注意__, _filter(filter(function, iterable))_, 如果 _function_ 为 _None_, 等价于生成器推导式 _(item for item in iterable if function(item))_. 如果 _function_ 不为 _None_, 等价于 _(item for item in iterable if item)_.

在 __itertools.filterfalse(predicate, iterable)__ 函数中提供了对内置函数 _filter_ 的补充扩展, 可以返回 _predicate_ 结果为_ False_ 的元素列表.

```python
# itertools.filterfalse 实现如下

def filterfalse(predicate, iterable):
    # filterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8
    if predicate is None:
        predicate = bool
    for x in iterable:
        if not predicate(x):    # 如果 predicate 结果为 False, 返回 x
            yield x
```

Map
---

> map(function, iterable, ...)

将可迭代对象 _iterable_ 的每一个元素都应用的到 _function_ 中, 并返回一个生成器, 生成结果集. 如果有另外的可迭代对象作为参数, 那么 _function_ 必须也要添加一个参数位置, 并迭代执行所有的参数. 当使用多个参数进行迭代时, 迭代次数参照最短可迭代对象参数的长度. 

```python
a = [1, 2, 3, 4, 5]
b = [10, 20, 30]

list(map(lambda x,y: x+y, a, b))
# Output: [11, 22, 33]

```

__注意__, 如果 _function_ 的参数是元组的列表集合, 可以参照 __itertools.starmap()__.


```python
# itertools.starmap 实现如下

def starmap(function, iterable):
    # starmap(pow, [(2,5), (3,2), (10,3)]) --> 32 9 1000
    for args in iterable:
        yield function(*args)
```

_map()_ 与 _itertools.starmap()_ 之间的区别在于传入 _function_ 参数是否被预先压缩. 

```python
from itertools import starmap

# 下面两个函数使用是等价的
list(map(lambda x,y: x+y, a, b))            # Output: [11, 22, 33]
list(starmap(lambda x,y: x+y, zip(a, b)))   # Output: [11, 22, 33]

```

Reduce
------

> functools.reduce(function, iterable[, initializer ])

__reduce__ 函数在 _Python2.7_ 中属于内置函数, 而在 _Python3.x_ 中被放到 __functools__ 中. 

_iterable_ 的每一个元素从左到右累积的应用于 _function_ 的两个参数中, 直到队列中只有一个值为止. 

```python
# For Example: 

reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])
# 运算过程等价于 ==>  ((((1+2)+3)+4)+5)
```
左边的参数 _x_, 是函数计算的累计结果, 右边的参数 _y_, 是 _iterable_ 序列中新的元素值. 如果 _initializer_ 初始项不为 _None_, 则将 _initializer_ 放在计算序列的最前面, 如果 _iterable_ 序列为空, 则被设置为默认值. 如果 _initializer_ 没有值, 且 _iterable_ 序列只有一个元素, 则返回第一个元素.

```python
# functools.reduce 实现如下

def reduce(function, iterable, initializer=None):
    it = iter(iterable)
    if initializer is None:
        value = next(it)
    else:
        value = initializer
    for element in it:
        value = function(value, element)
    return value
```