---
title: 'Python3-Cookbook第二章:字符串和文本(2)'
tags:
  - - Python
  - - 字符串
  - - Cookbook
categories:
  - [学习笔记, Python]
abbrlink: dae8b75d
date: 2020-11-16 19:25:44
---

#### 2.10 在正则式中使用Unicode

```python
# 在正则式中使用Unicode

""" 
你可以使用Unicode字符对应的转义序列(比如 \uFFF 或者 \UFFFFFFF )
"""
import re
arabic = re.compile('[\u0600-\u06ff\u0750-\u077f\u08a0-\u08ff]+')

```

#### 2.11 删除字符串中不需要的字符

```python
# 删除字符串中不需要的字符

""" 
strip() 方法能用于删除开始或结尾的字符。 
lstrip() 和 rstrip() 分别从左和从右执行删除操作。 
默认情况下，这些方法会去除空白字符，但是你也可以指定其他字符。
"""

# Whitespace stripping
s = ' hello world \n'
s.strip()  # 'hello world'
s.lstrip()  # 'hello world \n'
s.rstrip()  # ' hello world'

# Character stripping
t = '-----hello====='
t.lstrip('-')  # 'hello====='
t.strip('-=')  # 'hello'

```

#### 2.12 审查清理文本字符串

```python
# 审查清理文本字符串

""" 
除了使用str.upper() 和 str.lower()变换大小写格式。
str.replace() 或者 re.sub() 可以进行简单的替换操作。
还可以使用 str.translate() 方法，即自己创造映射之后进行替换。
"""
s = 'pýtĥöñ\fis\tawesome\r\n'  # 'pýtĥöñ\x0cis\tawesome\r\n'

# 映射
remap = {ord('\t'): ' ', ord('\f'): ' ', ord('\r'): None}
# 替换
a = s.translate(remap)  # 'pýtĥöñ is awesome\n'

```

#### 2.13 字符串对齐

```python
# 字符串对齐

""" 
对于基本的字符串对齐操作，可以使用字符串的 ljust() , rjust() 和 center() 方法。
所有这些方法都能接受一个可选的填充字符。
"""
text = 'Hello World'
text.ljust(20)  # 'Hello World         '
text.rjust(20)  # '         Hello World'
text.center(20)  # '    Hello World     '
# 有填充字符
text.rjust(20, '=')  # '=========Hello World'
text.center(20, '*')  # '****Hello World*****'

""" 
函数 format() 同样可以用来很容易的对齐字符串。 你要做的就是使用 <,> 或者 ^ 字符后面紧跟一个指定的宽度。
"""
format(text, '>20')  # '         Hello World'
format(text, '<20')  # 'Hello World         '
format(text, '^20')  # '    Hello World     '
'{:>10s} {:>10s}'.format('Hello', 'World')  # '     Hello      World'
# 有填充字符
format(text, '=>20s')  # '=========Hello World'
format(text, '*^20s')  # '****Hello World*****'

```

#### 2.14 合并拼接字符串

```python
# 合并拼接字符串

""" 
如果字符串在一个list里或者iterable中，使用join()
"""
parts = ['Is', 'Chicago', 'Not', 'Chicago?']
' '.join(parts)  # 'Is Chicago Not Chicago?'
','.join(parts)  # 'Is,Chicago,Not,Chicago?'
''.join(parts)  # 'IsChicagoNotChicago?'

# 永远都不应该如下写
s = ''
for p in parts:
    s += p

# 适当的时候可以使用生成器表达式
','.join(str(d) for d in data)

# 输出的时候没必要将字符串连接
print(a, b, c, sep=':')  # Better

```

#### 2.15 字符串中插入变量

```python
# 字符串中插入变量

""" 
可以通过字符串的format()方法来解决。
"""
s = '{name} has {n} messages.'
s.format(name='Guido', n=37)  # 'Guido has 37 messages.'

""" 
如果要被替换的变量能在变量域中找到， 那么你可以结合使用 format_map() 和 vars()。
"""
name = 'Guido'
n = 37
s.format_map(vars())  # 'Guido has 37 messages.'

```

#### 2.16 以指定列宽格式化字符

```python
# 以指定列宽格式化字符

""" 
使用 textwrap 模块来格式化字符串的输出
"""
import os
import textwrap
s = "Look into my eyes, look into my eyes, the eyes, the eyes, \
the eyes, not around the eyes, don't look around the eyes, \
look into my eyes, you're under."
# 输出以多少字符换行
print(textwrap.fill(s, 70))
print(textwrap.fill(s, 40))
# 首行缩进
print(textwrap.fill(s, 40, initial_indent='    '))
# 次行及之后缩进
print(textwrap.fill(s, 40, subsequent_indent='    '))

# 获取终端大小尺寸
os.get_terminal_size().columns

```

#### 2.17 在字符串中处理html和xml

```python
# 在字符串中处理html和xml

""" 
使用html模块中的函数
"""
import html
s = 'Elements are written as "<tag>text</tag>".'
# 替换字符串中的尖括号
print(html.escape(s))
# Elements are written as &quot;&lt;tag&gt;text&lt;/tag&gt;&quot;.

# 将非ASCII文本对应的编码实体嵌入进去
s = 'Spicy Jalapeño'
s.encode('ascii', errors='xmlcharrefreplace')
# b'Spicy Jalape&#241;o'

# 使用HTML或者XML解析器的一些相关工具函数/方法

s = 'Spicy &quot;Jalape&#241;o&quot.'
from html.parser import HTMLParser
p = HTMLParser()
p.unescape(s)
# 'Spicy "Jalapeño".'
t = 'The prompt is &gt;&gt;&gt;'
from xml.sax.saxutils import unescape
unescape(t)
# 'The prompt is >>>'

```

#### 2.18 字符串令牌解析

```python
# 字符串令牌解析

"""  
模式对象scanner() 方法。 
这个方法会创建一个 scanner 对象， 在这个对象上不断的调用 match() 方法会一步步的扫描目标文本，每步一个匹配。
"""
# 想将t转换为tokens
import re
t = 'foo = 23 + 42 * 10'
tokens = [('NAME', 'foo'), ('EQ', '='), ('NUM', '23'), ('PLUS', '+'),
          ('NUM', '42'), ('TIMES', '*'), ('NUM', '10')]

NAME = r'(?P<NAME>[a-zA-Z_][a-zA-Z_0-9]*)'
NUM = r'(?P<NUM>\d+)'
PLUS = r'(?P<PLUS>\+)'
TIMES = r'(?P<TIMES>\*)'
EQ = r'(?P<EQ>=)'
WS = r'(?P<WS>\s+)'

master_pat = re.compile('|'.join([NAME, NUM, PLUS, TIMES, EQ, WS]))

scanner = master_pat.scanner('foo = 42')
scanner.match()
# <_sre.SRE_Match object at 0x100677738>
_.lastgroup, _.group()
# ('NAME', 'foo')
scanner.match()
# <_sre.SRE_Match object at 0x100677738>
_.lastgroup, _.group()
# ('WS', ' ')
scanner.match()
# <_sre.SRE_Match object at 0x100677738>
_.lastgroup, _.group()
# ('EQ', '=')
scanner.match()
# <_sre.SRE_Match object at 0x100677738>
_.lastgroup, _.group()
# ('WS', ' ')
scanner.match()
# <_sre.SRE_Match object at 0x100677738>
_.lastgroup, _.group()
# ('NUM', '42')
scanner.match()

```

---

> 参考：
>
> https://python3-cookbook.readthedocs.io/zh_CN/latest/
>
> https://github.com/yidao620c/python3-cookbook