> 读取 .conf 文件配置信息

<!--more-->

Python代码

```python
################## Config.py ##################
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import configparser


class Config(object):
    """
    python3 读取 ini 格式配置文件
    
    >>> from Config import Config
    >>> http_addr = Config().get('listen', 'HttpAddr')
    >>> http_port = Config().get('listen', 'HttpPort')
    >>> sql_echo = Config().get('sqlalchemy', 'Echo')
    >>> print(http_addr, http_port, sql_echo)
    0.0.0.0 5005 false
    
    """

    _instance = None

    def _init(self):
        """自定义初始化方法"""
        self._config = configparser.ConfigParser()
        with open('app.conf', 'r', encoding='utf-8') as fp:
            ini_string = fp.read()
            fp.close()
            self._config.read_string(ini_string)

    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super(Config, cls).__new__(cls, *args, **kwargs)
            cls._init(cls)
        return cls._instance

    def get(self, field, key):
        try:
            result = self._config.get(field, key)
        except:
            result = ""
        return result

    def sections(self):
        return self._config.sections()
```

配置文件 app.conf

```
### Server ###

[listen]
HttpAddr = 0.0.0.0
HttpPort = 5005

[sqlalchemy]
### SQLAlchemy ###
Echo = false
```
