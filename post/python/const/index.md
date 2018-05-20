> 在Python开发过程中，经常会有用到不可变常量的时候，但一般都是直接定义Python数据类型。但这并不安全，在以后的编码过程中，我们很有可能在自己不知道的情况下修改这个变量的值，但事实上并不希望如此。

<!--more-->

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

class _const:
    """
    Defines a constant in Python that throws an exception
    when changing the value of a constant or when the constant name is not capitalized. 
    """
    class ConstError(TypeError):pass
    class ConstCaseError(ConstError): pass

    def __setattr__(self, name, value):
        if name in self.__dict__:
            raise self.ConstError("can't change const %s" % name)
        if not name.isupper():
            raise self.ConstCaseError('const name "%s" is not all uppercase' % name)
        self.__dict__[name] = value

# import sys
# sys.modules[__name__] = _const()

const = _const()


# Examples:
"""
const.NUMBER_0 = 0      # Ture
const.INFO = 'info'     # Ture

const.number_1 = 1      # False ConstCaseError: const name 'number_1' is not all uppercase

const.NUMBER_0 = '0'    # False ConstError: can't change const 'NUMBER_0'
"""
```