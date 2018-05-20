> 让目录或者zip文件成为可运行的脚本

> 在书写python脚本的时候，往往会使用不止一个 python 代码文件，而将多个文件放在一个目录下或者zip包中，使目录或者zip包可以直接运行，这样使得多个python简脚本运行变得非常方便

<!--more-->

```python
# 首先在多个python文件所在的目录中添加 __main__.py 文件，作为简本执行的入口
# 文件目录结构如下:
app/
  ├── __main__.py
  ├── a.py
  ├── b.py
  └── c.py

#### #
# __main__.py
#### #

# !/usr/bin/env python3
# -*- coding: utf-8 -*-

import a
import b
import c

if __name__ == "__main__":
    print('from __main__.py')

#### #
# a.py b.py c.py 文件内容基本相同
#### #

print('from a.py')

// 运行示例代码
shell> python3 app
from a.py
from b.py
from c.py
from __main__.py

// 打包zip
shell> zip -r app.zip *.py
shell> python3 app.zip
from a.py
from b.py
from c.py
from __main__.py

# 注：
# python中还有其他将python程序打包成可执行文件的方式, 如 pyinstaller
```

-----------

> 根据日期字符串,返回所在周全部日期

```python
from datetime import datetime, time, timedelta

def getWeeksByDay(day):
    me = datetime.strptime(day, "%Y-%m-%d")
    week = me.weekday()
    return [(me + timedelta(days=d)).strftime("%Y-%m-%d") for d in range(-week, 7 - week)]

getWeeksByDay("2018-01-05")
"""
['2018-01-01',
 '2018-01-02',
 '2018-01-03',
 '2018-01-04',
 '2018-01-05',
 '2018-01-06',
 '2018-01-07']
"""
```
