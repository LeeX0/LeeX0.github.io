---
title: 'Python3-Cookbook第二章:字符串和文本(1)'
tags:
  - - Python
  - - 字符串
  - - Cookbook
categories:
  - [学习笔记, Python]
abbrlink: f1c5e49e
date: 2020-11-14 19:25:41
---

#### 2.1 使用多个界定符分割字符串

```python
# 使用多个界定符分割字符串

""" 
string 对象的 split() 方法只适应于非常简单的字符串分割情形。
re.split() 方法可以更加灵活的切割字符串。
"""
import re
line = 'asdf fjdk; afed, fjek,asdf, foo'
# 表示可以通过;或,或空格或多个空格分隔
re.split(r'[;,\s]\s*', line)
# Outputs
# ['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']

```

#### 2.2 字符串开头或结尾匹配

```python
# 字符串开头或结尾匹配

""" 
简单使用可以通过str.startswith() 或者是 str.endswith() 方法。
如果需要多种匹配可以将匹配项作为元组（且必须是元组tuple()）传入上述方法。
切片实现不优雅，正则实现较复杂，这种方式简单使用较方便。
"""
filename = 'spam.txt'
filename.endswith('.txt')  # True
filename.startswith('file:')  # False
url = 'http://www.python.org'
url.startswith('http:')  # True

[name for name in filenames if name.endswith(('.c', '.h'))]

```

#### 2.3 用Shell通配符匹配字符串

```python
# 用Shell通配符匹配字符串

""" 
fnmatch 模块提供了两个函数—— fnmatch() 和 fnmatchcase() ，可以用来实现这样的匹配。
fnmatch()大小写敏感根据系统不同而不尽相同。fnmatchcase()大小写敏感。
功能强度介于字符串与正则之间。
"""
from fnmatch import fnmatch, fnmatchcase
fnmatch('foo.txt', '*.txt')  # True
fnmatch('foo.txt', '?oo.txt')  # True
fnmatch('Dat45.csv', 'Dat[0-9]*')  # True

```

#### 2.4 字符串匹配和搜索

```python
# 字符串匹配和搜索

""" 
str.find() , str.endswith() , str.startswith() 或者类似的方法可做简单匹配。
复杂匹配可以使用re模块与正则表达式。表达式多次使用可以预编译为模式对象。
"""
import re
text1 = '11/27/2012'
text2 = 'Nov 27, 2012'
if re.match(r'\d+/\d+/\d+', text1):
    print('yes')
else:
    print('no')
if re.match(r'\d+/\d+/\d+', text2):
    print('yes')
else:
    print('no')

# 预编译为模式对象
# 字符串前的r表示字符串为raw string，即不会转义。如果此处没有r需要双反斜杠阅读性差。
datepat = re.compile(r'\d+/\d+/\d+')
if datepat.match(text1):
    print('yes')
else:
    print('no')

""" 
match() 总是从字符串开始去匹配，如果你想查找字符串任意部分的模式出现位置， 使用 findall() 方法去代替。
如果你想以迭代方式返回匹配，可以使用 finditer() 方法来代替。
"""
text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
datepat.findall(text)
# Outputs
# ['11/27/2012', '3/13/2013']

""" 
使用括号去捕获分组，分别将每个组的内容提取出来。
"""
datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
m = datepat.match('11/27/2012')
# <_sre.SRE_Match object at 0x1005d2750>
m.group(0)  # '11/27/2012'
m.group(1)  # '11'
m.group(2)  # '27'
m.group(3)  # '2012'
m.groups()  # ('11', '27', '2012')

# Find all matches (notice splitting into tuples)
text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
datepat.findall(text)
[('11', '27', '2012'), ('3', '13', '2013')]
for month, day, year in datepat.findall(text):
    print('{}-{}-{}'.format(year, month, day))

```

#### 2.5 字符串搜索和替换

```python
# 字符串搜索和替换

""" 
简单字面模式可以使用str.replace()。
对于复杂的模式，请使用 re 模块中的 sub() 函数。
"""
# str.replace()
import re
text = 'yeah, but no, but yeah, but no, but yeah'
text.replace('yeah', 'yep')
# 'yep, but no, but yep, but no, but yep'

# re.sub()
text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
# 'Today is 2012-11-27. PyCon starts 2013-3-13.'
# sub() 函数中的第一个参数是被匹配的模式，第二个参数是替换模式。反斜杠数字比如 \3 指向前面模式的捕获组号。

# 预编译
datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
datepat.sub(r'\3-\1-\2', text)

```

#### 2.6 字符串忽略大小写的搜索替换

```python
# 字符串忽略大小写的搜索替换

""" 
为了在文本操作时忽略大小写，你需要在使用 re 模块的时候给这些操作提供 re.IGNORECASE 标志参数。
"""
import re
text = 'UPPER PYTHON, lower python, Mixed Python'
re.findall('python', text, flags=re.IGNORECASE)
# ['PYTHON', 'python', 'Python']
re.sub('python', 'snake', text, flags=re.IGNORECASE)
# 'UPPER snake, lower snake, Mixed snake'

```

#### 2.7 最短模式匹配

```python
# 最短模式匹配

""" 
通过正则表达式的限定符?改变匹配的模式为最短匹配。
"(.*)"表示最长匹配双引号内的内容。
"(.*?)"表示最短匹配双引号内的内容。
"""
import re
str_pat = re.compile(r'"(.*)"')
text1 = 'Computer says "no."'
str_pat.findall(text1)
# ['no.']
text2 = 'Computer says "no." Phone says "yes."'
str_pat.findall(text2)
# ['no." Phone says "yes.']

str_pat = re.compile(r'"(.*?)"')
str_pat.findall(text2)
# ['no.', 'yes.']

```

#### 2.8 多行匹配模式

```python
# 多行匹配模式

""" 
当你用点(.)去匹配任意字符的时候，发现点(.)不能匹配换行符的事实。
其中一种场景就是匹配C语言的跨行注释。
"""
import re
# 该模式对象只能匹配未换行的注释
comment = re.compile(r'/\*(.*?)\*/')
# 修改模式增加对换行的支持
# (?:.|*?)中，?:表示匹配但不获取(非捕获组)，|表示或关系，*？表示任意数量但最短匹配
comment = re.compile(r'/\*((?:.|\n)*?)\*/')

```

#### 2.9 将Unicode文本标准化

```python
# 将Unicode文本标准化

""" 
使用unicodedata模块将文本标准化
"""
import unicodedata
s1 = 'Spicy Jalape\u00f1o'  # 'Spicy Jalapeño'
s2 = 'Spicy Jalapen\u0303o'  # 'Spicy Jalapeño'
s1 == s2  # False

t1 = unicodedata.normalize('NFC', s1)
t2 = unicodedata.normalize('NFC', s2)
t1 == t2  # True

```

---

> 参考：
>
> https://python3-cookbook.readthedocs.io/zh_CN/latest/
>
> https://github.com/yidao620c/python3-cookbook