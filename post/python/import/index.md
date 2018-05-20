> 使用字符串名称导入模块

<!--more-->

```python
#
# 在老版本的 python 中使用内置方法 __import__ 导入模块
# import_class 方法实现了从字符串导入自定义的类，并实例化调用的过程

################## import_class.py ##################
"""
Dynamic import module.
    # Foo class in foo.py.
    # Module's name is foo.

    # class Foo(object):
    #     def __init__(self):
    #         pass
    #
    #     def run(self):
    #         print("print Foo.run")

    >>> module_str = 'foo.foo.Foo'
    >>> Foo = import_class(module_str)
    >>> Foo().run() # print: print Foo.run
"""

import sys
import traceback


def import_class(import_str):
    """Returns a class from a string including module and class. """
    
    mod_str, _sep, class_str = import_str.rpartition('.')
    __import__(mod_str)
    try:
        return getattr(sys.modules[mod_str], class_str)
    except AttributeError:
        raise ImportError('Class %s cannot be found (%s)' %
                          (class_str,
                           traceback.format_exception(*sys.exc_info())))


def import_object(import_str, *args, **kwargs):
    """Import a class and return an instance of it.
    """
    return import_class(import_str)(*args, **kwargs)


def import_module(import_str):
    """Import a module.
    """
    __import__(import_str)
    return sys.modules[import_str]


#
# 在新版本的 python 中使用 importlib 模块实现导入
# 当模块或包的名称以自负串的形式给出时，可以使用 importlib.import_module() 函数来手动导入这个模块
# 上面的 __import__ 内置函数调用可以替换为 importlib.import_module()，且更加方便

################## __import__ ##################
def import_module(import_str):
    """Import a module.
    """
    __import__(import_str)
    return sys.modules[import_str]

################## importlib ##################
import importlib

def import_module_str(import_str):
	return importlib.import_module(import_str)
	
# 上面的 import_class 可以替换为

def import_class(import_str):
    """Returns a class from a string including module and class.
    """
    mod_str, _sep, class_str = import_str.rpartition
    try:
    	# 省略 sys 调用
        return getattr(importlib.import_module(mod_str), class_str)
    except AttributeError:
        raise ImportError('Class %s cannot be found (%s)' %
                          (class_str,
                           traceback.format_exception(*sys.exc_info())))

# 相对导入

import importlib

# 等同于  `from . import foo`
b = importlib.import_module('.foo', __package__)


```
