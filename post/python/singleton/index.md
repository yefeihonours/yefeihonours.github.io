> 使用装饰器实现单例模式

<!--more-->

```python
def singleton(aClass):
    """ 
    使用函数定义的装饰器, 每一个被装饰类都拥有同一个 singleton 函数对象
    >>> @singleton
    >>> class SingletonClass:
    >>>     pass
    >>> ins1, ins2 = SingletonClass(), SingletonClass()
    >>> id(ins1) == id(ins2)
    True

    """
    def onCall(*args, **kwargs):
        if onCall.instance is None:
            onCall.instance = aClass(*args, **kwargs)
        return onCall.instance
    onCall.instance = None
    return onCall


class singleton(object):
    """
    使用类定义的装饰器, 每一个被装饰类都拥有同一个 singleton 对象实例
    >>> @singleton
    >>> class SingletonClass:
    >>>     pass
    >>> ins1, ins2 = SingletonClass(), SingletonClass()
    >>> id(ins1) == id(ins2)
    True
    
    """

    def __init__(self, aClass):
        self.aClass = aClass
        self.instance = None
    
    def __call__(self, *args, **kwargs):
        if self.instance is None:
            self.instance = self.aClass(*args, **kwargs)
        return self.instance
```
