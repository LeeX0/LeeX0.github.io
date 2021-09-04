---
title: 'Python3-Cookbook第一章:数据结构和算法(2)'
tags:
  - - Python
  - - 数据结构
  - - 算法
  - - Cookbook
categories:
  - [学习笔记, Python]
abbrlink: d590dae
date: 2020-11-07 16:45:41
---

#### 1.11 命名切片

```python
# 命名切片
# 假定你要从一个记录（比如文件或其他类似格式）中的某些固定位置提取字段

# 与其
record = '....................100 .......513.25 ..........'
cost = int(record[20:23]) * float(record[31:37])

# 不如
SHARES = slice(20, 23)
PRICE = slice(31, 37)
cost = int(record[SHARES]) * float(record[PRICE])

```

#### 1.12 序列中出现次数最多的元素

```python
# 序列中出现次数最多的元素

""" 
collections.Counter 类就是专门为这类问题而设计的， 它甚至有一个有用的 most_common() 方法直接给了你答案。
"""
# 取出出现频率最高的单词
from collections import Counter
words = [
    'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
    'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not', 'around', 'the',
    'eyes', "don't", 'look', 'around', 'the', 'eyes', 'look', 'into',
    'my', 'eyes', "you're", 'under'
]
word_counts = Counter(words)
# 出现频率最高的3个单词
top_three = word_counts.most_common(3)
print(top_three)
# Outputs [('eyes', 8), ('the', 5), ('look', 4)]

""" 
collections.Counter底层实际上是一个元素作为key，出现次数作为value的dict。
神奇的是还能进行数学运算操作结合。
"""
morewords = ['why','are','you','not','looking','in','my','eyes']
a = Counter(words)
b = Counter(morewords)
a
# Outputs
# Counter({'eyes': 8, 'the': 5, 'look': 4, 'into': 3, 'my': 3, 'around': 2,"you're": 1, "don't": 1, 'under': 1, 'not': 1})
b
# Outputs
# Counter({'eyes': 1, 'looking': 1, 'are': 1, 'in': 1, 'not': 1, 'you': 1,'my': 1, 'why': 1})

# Combine counts
c = a + b
c
# Outputs
# Counter({'eyes': 9, 'the': 5, 'look': 4, 'my': 4, 'into': 3, 'not': 2,'around': 2, "you're": 1, "don't": 1, 'in': 1, 'why': 1,'looking': 1, 'are': 1, 'under': 1,'you': 1})

# Subtract counts
d = a - b
d
# Outputs
# Counter({'eyes': 7, 'the': 5, 'look': 4, 'into': 3, 'my': 2, 'around': 2,"you're": 1, "don't": 1, 'under': 1})

```

#### 1.13 通过某个关键字排序一个字典列表

```python
# 通过某个关键字排序一个字典列表

""" 
使用 operator 模块的 itemgetter 函数。
排序过程中，相当于key使用的itemgetter获取到了dict中对应key的value进行排序。
itemgetter同时支持多个keys。
"""
from operator import itemgetter
rows = [
    {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003},
    {'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
]
rows_by_fname = sorted(rows, key=itemgetter('fname'))
rows_by_uid = sorted(rows, key=itemgetter('uid'))
print(rows_by_fname)
print(rows_by_uid)

rows_by_lfname = sorted(rows, key=itemgetter('lname', 'fname'))
print(rows_by_lfname)

""" 
在上面例子中， rows 被传递给接受一个关键字参数的 sorted() 内置函数。 
这个参数是 callable 类型，并且从 rows 中接受一个单一元素，然后返回被用来排序的值。
itemgetter() 函数就是负责创建这个 callable 对象的。
本例中的操作基本等同于key中使用lambda定义，但是效率更高。同样适用于max，min等。
"""
rows_by_fname = sorted(rows, key=lambda r: r['fname'])
rows_by_lfname = sorted(rows, key=lambda r: (r['lname'], r['fname']))

```

#### 1.14 排序不支持原生比较的对象

```python
# 排序不支持原生比较的对象

""" 
关键在于传入callable的key参数时，获取到非原生对象的具体变量。
可以通过lambda或者operator中的attrgetter。
"""
from operator import attrgetter


class User:
    def __init__(self, user_id):
        self.user_id = user_id

    def __repr__(self):
        return 'User({})'.format(self.user_id)


users = [User(23), User(3), User(99)]
# Method 1
print(sorted(users, key=lambda u: u.user_id))

# Method 2
print(sorted(users, key=attrgetter('user_id')))

```

#### 1.15 通过某个字段将记录分组

```python
# 通过某个字段将记录分组

""" 
使用itertools.groupby()函数，务必记得需要先排序
"""
from collections import defaultdict
from itertools import groupby
from operator import itemgetter
rows = [
    {'address': '5412 N CLARK', 'date': '07/01/2012'},
    {'address': '5148 N CLARK', 'date': '07/04/2012'},
    {'address': '5800 E 58TH', 'date': '07/02/2012'},
    {'address': '2122 N CLARK', 'date': '07/03/2012'},
    {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'},
    {'address': '1060 W ADDISON', 'date': '07/02/2012'},
    {'address': '4801 N BROADWAY', 'date': '07/01/2012'},
    {'address': '1039 W GRANVILLE', 'date': '07/04/2012'},
]

# 需要预先按照想要分组的item排序
rows.sort(key=itemgetter('date'))
# 分组
for date, items in groupby(rows, key=itemgetter('date')):
    print(date)
    for i in items:
        print(' ', i)
""" 输出结果
07/01/2012
  {'date': '07/01/2012', 'address': '5412 N CLARK'}
  {'date': '07/01/2012', 'address': '4801 N BROADWAY'}
07/02/2012
  {'date': '07/02/2012', 'address': '5800 E 58TH'}
  {'date': '07/02/2012', 'address': '5645 N RAVENSWOOD'}
  {'date': '07/02/2012', 'address': '1060 W ADDISON'}
07/03/2012
  {'date': '07/03/2012', 'address': '2122 N CLARK'}
07/04/2012
  {'date': '07/04/2012', 'address': '5148 N CLARK'}
  {'date': '07/04/2012', 'address': '1039 W GRANVILLE'}
"""

""" 
如果需要保持序列顺序，则可以通过1-6中的defaultdict，将想要分组的item作为其中的key，然后将整条记录append到对应item的组别中。
"""
rows_by_date = defaultdict(list)
for row in rows:
    rows_by_date[row['date']].append(row)

```

#### 1.16 过滤序列元素

```python
# 过滤序列元素

""" 
可通过列表推导式达到目的，好处是同时还能充当简单的数据住转换。但是当元素结果集很大时则很占内存。
可以将过滤代码放到一个函数中， 然后使用内建的 filter() 函数。
"""
# filter得到的是一个迭代器，如果想得到列表还需要进行list()转换
values = ['1', '2', '-3', '-', '4', 'N/A', '5']

def is_int(val):
    try:
        x = int(val)
        return True
    except ValueError:
        return False

ivals = list(filter(is_int, values))
print(ivals)
# Outputs ['1', '2', '-3', '4', '5']

""" 
列表推导式
1. [i for i in range(k) if condition]：此时if起条件判断作用，满足条件的，将被返回成为最终生成的列表的一员。
2. [i if condition else exp for exp]：此时if...else被用来赋值，满足条件的i以及else被用来生成最终的列表。
"""
print([i for i in range(10) if i % 2 == 0])
print([i if i == 0 else 100 for i in range(10)])
# Outputs
[0, 2, 4, 6, 8]
[0, 100, 100, 100, 100, 100, 100, 100, 100, 100]

```

#### 1.17 从字典中提取子集

```python
# 从字典中提取子集

# 使用字典推导
prices = {
    'ACME': 45.23,
    'AAPL': 612.78,
    'IBM': 205.55,
    'HPQ': 37.20,
    'FB': 10.75
}
# 取出value大于200的
p1 = {key: value for key, value in prices.items() if value > 200}
# 取出在name列表中的键值对
tech_names = {'AAPL', 'IBM', 'HPQ', 'MSFT'}
p2 = {key: value for key, value in prices.items() if key in tech_names}

```

#### 1.18 映射名称到序列元素

```python
# 映射名称到序列元素

""" 
使用collections.namedtuple()，函数使用即在初始化的时候传入一个类型名与需要的下标字段。
通过使用下标对元组中的内容进行组合操作会表意不清晰。
命名元组与字典功能很接近，但是要更节省内存。
"""
from collections import namedtuple
Subscriber = namedtuple('Subscriber', ['addr', 'joined'])
sub = Subscriber('jonesy@example.com', '2012-10-19')
print(sub)  # Subscriber(addr='jonesy@example.com', joined='2012-10-19')
print(sub.addr)  # jonesy@example.com
print(sub.joined)  # 2012-10-19

# 举例使用，在命名元组中使用下标。
# 实际上本身对records中的每条记录内容都知道是什么，只是为了表意清楚，使用有名字的下标进行计算。
Stock = namedtuple('Stock', ['name', 'shares', 'price'])

def compute_cost(records):
    total = 0.0
    for rec in records:
        s = Stock(*rec)
        total += s.shares * s.price
    return total

```

#### 1.19 转换并同时计算数据

```python
# 转换并同时计算数据

""" 
练习使用生成器表达式参数
"""
# 计算平方和
import os
nums = [1, 2, 3, 4, 5]
s = sum(x * x for x in nums)

# 查看文件夹中是否包含.py后缀文件
files = os.listdir('dirname')
if any(name.endswith('.py') for name in files):
    print('There be python!')
else:
    print('Sorry, no python.')
# 计算字典列表中某个key最小的value
portfolio = [
    {'name': 'GOOG', 'shares': 50},
    {'name': 'YHOO', 'shares': 75},
    {'name': 'AOL', 'shares': 20},
    {'name': 'SCOX', 'shares': 65}
]
min_shares = min(s['shares'] for s in portfolio)

# 对于min() 和 max() 它们接受的一个 key 关键字参数或许对你很有帮助

```

#### 1.20 合并多个字典或映射

```python
# 合并多个字典或映射

""" 
使用collections 模块中的 ChainMap 类
"""
from collections import ChainMap
a = {'x': 1, 'z': 3}
b = {'y': 2, 'z': 4}
c = ChainMap(a, b)  # 先从a找，再从b找
print(c['x'])  # Outputs 1 (from a)
print(c['y'])  # Outputs 2 (from b)
print(c['z'])  # Outputs 3 (from a)

""" 
通过这种操作的字典并不是真正的合并了，只是内部创建了容纳这些字典的列表，大部分字典操作可以正常使用。
对于新字典的更新与删除会影响列表中的第一个字典。而使用update()方法，原字典的更新不会影响到新的合并字典。
"""
a = {'x': 1, 'z': 3}
b = {'y': 2, 'z': 4}
merged = dict(b)
merged.update(a)
merged['x']  # 1
merged['y']  # 2
merged['z']  # 3

a['x'] = 13
merged['x']  # 1

```

---

> 参考：
>
> https://python3-cookbook.readthedocs.io/zh_CN/latest/
>
> https://github.com/yidao620c/python3-cookbook