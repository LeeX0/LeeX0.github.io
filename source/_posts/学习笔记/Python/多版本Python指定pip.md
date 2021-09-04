---
title: 多版本Python指定pip
tags:
  - - Python
  - - pip
categories:
  - - 学习笔记
    - Python
abbrlink: 741c6074
date: 2021-07-07 16:20:08
---

> 因为系统中有时需要使用多个版本的Python，对应可能有多个版本的pip。
>
> 在指定与更新过程中之前有点没搞清楚，简单记录。

## 安装

如果`Ubuntu`中已经安装了`Python2`和`Python3`。可以通过脚本安装对应的`pip`。

```bash
# 为Python3安装pip
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --force-reinstall

# 为Python2安装pip
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
python get-pip.py --force-reinstall
```

若要为具体版本安装pip将`python3 get-pip.py --force-reinstall`中的`python3`换成如`python3.5`或者`python3.6`之类的即可。

通过`pip -V`或`pip3 -V`即可查看对应的版本。

## 管理

在安装使用过程中可能产生`pip`没有对应到`python2`或者`pip3`没有对应`python3`之类的问题。

可以在`/usr/local/bin`目录中看到，有很多`pip`相关的脚本，`pip`、`pip2`、`pip3`等。实际内容都是`python`脚本。

**例如需要将`pip3`对应到`python3`，将脚本第一行的解释器对应到需要的`python`版本即可。**

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
import re
import sys

from pip._internal import main

if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
```

如果出现`ImportError: No module named _internal`错误，可能是你修改后的解释器还没有安装`pip`包，使用前面所述方法进行安装。

*另外：直接改这个脚本文件来修改pip对应的python版本的前提是，当前脚本文件对应的Python版本与要修改对应的Python版本的pip包版本最好一致，不一致很可能出错。*

---

> 参考：
>
> https://zhuanlan.zhihu.com/p/37473690
>
> https://bootstrap.pypa.io/
