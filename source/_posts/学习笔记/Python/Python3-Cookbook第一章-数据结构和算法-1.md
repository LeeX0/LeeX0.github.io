---
title: 'Python3-Cookbook第一章:数据结构和算法(1)'
tags:
  - - Python
  - - 数据结构
  - - 算法
  - - Cookbook
categories:
  - [学习笔记, Python]
abbrlink: 26745e6d
date: 2020-11-05 16:30:01
---

> 对于**《Python Cookbook》**一书进行学习，通过书中部分代码注释与自己的理解写成笔记方便学习与回忆。

#### 1.1 解压序列赋值给多个变量

```python
# 解压序列赋值给多个变量
data = ['ACME', 50, 91.1, (2012, 12, 21)]
name, shares, price, date = data
print("name: {}".format(name))  # Prints name: ACME
print("shares: {}".format(shares))  # Prints shares: 50
print("price: {}".format(price))  # Prints price: 91.1
print("data: {}".format(data))  # Prints data: ['ACME', 50, 91.1, (2012, 12, 21)]

# 不需要的变量用占位符取代，最后丢弃即可
name, _, _, date = data

# 字符串也可通过此种方式取值
s = "hello"
a, b, c, d, e = s
print("c: {}".format(c))  # Prints c: l

```

#### 1.2 解压可迭代对象赋值给多个变量

```python
# 解压可迭代对象赋值给多个变量
# 1-1中如果item数量需要确定的去取

""" 
Python中的星号键不是指针，代表取一个不定长的list；同样两个星号代表取一个不定长的dict 
"""
record = ('Dave', 'dave@example.com', '773-555-1212', '847-555-1212')
name, email, *number = record
print("name: {}".format(name))  # Prints name: Dave
print("email: {}".format(email))  # Prints email: dave@example.com
print("number: {}".format(number))  # Prints number: ['773-555-1212', '847-555-1212']

# 用此种方法可以轻易取list中的第一个元素与最后一个元素
record = ('Dave', 'dave@example.com', '773-555-1212', '847-555-1212', 'boss11')
name, *_ = record
*_, stuff = record

# 同样此方法可以用来获取不确定的参数，用以函数传参等
records = [('foo', 1, 2), ('bar', 'hello'), ('foo', 3, 4), ]

def do_foo(x, y):
    print('foo', x, y)

def do_bar(s):
    print('bar', s)

for tag, *args in records:
    if tag == 'foo':
        do_foo(*args)
    elif tag == 'bar':
        do_bar(*args)

# Prints 
# foo 1 2
# bar hello
# foo 3 4
```

#### 1.3 保留最后N个元素

```python
# 保留最后N个元素

""" 
使用collections.deque即可完成该操作，deque是一个可以设置长度的双端队列。
具有append(),appendleft(),pop(),popleft()。
"""
from collections import deque

q = deque(maxlen=3)
q.append(1)
q.append(2)
q.append(3)
print(q)  # Prints deque([1, 2, 3], maxlen=3)
q.append(4)
print(q)  # Prints deque([2, 3, 4], maxlen=3)

# 不指定大小则获得一个无限长度的双端队列
q = deque()
q.append(1)
q.append(2)
q.append(3)
print(q)  # Prints deque([1, 2, 3])
q.appendleft(4)
print(q)  # Prints deque([4, 1, 2, 3])
print(q.pop())  # Prints 3
print(q.popleft())  # Prints 4

# 举例：保留文件中包含python字样的最后五行数据
def search(lines, pattern, history=5):
    previous_lines = deque(maxlen=history)
    for line in lines:
        if pattern in line:
            yield line, previous_lines
        previous_lines.append(line)

if __name__ == '__main__':
    with open(r'./somefile.txt') as f:
        for line, prevlines in search(f, 'python', 5):
            for pline in prevlines:
                print(pline, end='')
                print(line, end='')
                print('-' * 20)

```

#### 1.4 查找最大最小的N个元素

```python
# 查找最大最小的N个元素

""" 
heapq 模块有两个函数：nlargest() 和 nsmallest() 可以进行输出
"""
import heapq

nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
print(heapq.nlargest(3, nums))  # Prints [42, 37, 23]
print(heapq.nsmallest(3, nums))  # Prints [-4, 1, 2]

# 同时支持复杂的数据结构的排序,需要传入一个key函数
portfolio = [
    {'name': 'IBM', 'shares': 100, 'price': 91.1},
    {'name': 'AAPL', 'shares': 50, 'price': 543.22},
    {'name': 'FB', 'shares': 200, 'price': 21.09},
    {'name': 'HPQ', 'shares': 35, 'price': 31.75},
    {'name': 'YHOO', 'shares': 45, 'price': 16.35},
    {'name': 'ACME', 'shares': 75, 'price': 115.65}
]
cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])

# 使用heapq.heapify对list进行处理
heap = list(nums)
heapq.heapify(heap)  # 对heap进行排序
heapq.heappop(heap)  # 弹出heap最下的元素，根据堆的性质每次都会弹出最小

```

#### 1.5 优先级队列

```python
# 优先级队列，与一个元组顺序比较的小问题

""" 
利用1-4中的heapq堆，将优先级的负数传入，根据heapq每次会pop最小值的属性，从而每次pop出优先级最高的item（同样优先级的元素按照插入顺序输出）
"""
import heapq


class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0

    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, self._index, item))
        self._index += 1

    def pop(self):
        return heapq.heappop(self._queue)[-1]


""" 
其中index变量的作用是保证同等优先级元素的正确排序。 通过保存一个不断增加的index下标变量，可以确保元素按照它们插入的顺序排序。 
而且index变量也在相同优先级元素比较的时候起到重要作用。 
"""
# 展示另一个多元组大小比较的问题，多元组是依次比较同位置的元素，上例中的index就是避免了对比错误
a = (1, Item('foo'))
b = (5, Item('bar'))
a < b  # Print True
c = (1, Item('grok'))
a < c  # Print TypeError

```

#### 1.6 字典中的键映射多个值

```python
# 字典中的键映射多个值
from collections import defaultdict

d = defaultdict(list)
d['a'].append(1)
d['a'].append(2)
d['b'].append(4)

d = defaultdict(set)
d['a'].add(1)
d['a'].add(2)
d['b'].add(4)

""" 
元组tuple(),不可变有序
列表list[],可变有序
字典dict{key:value},存键值对无序
集合set{},无重复无序
"""

```

#### 1.7 字典排序

```python
# 字典排序

""" 
collections中的OrderedDict可以按照键值对插入字典的顺序存储dict
"""
from collections import OrderedDict

d = OrderedDict()
d['foo'] = 1
d['bar'] = 2
d['spam'] = 3
d['grok'] = 4
# Outputs "foo 1", "bar 2", "spam 3", "grok 4"
for key in d:
    print(key, d[key])

""" 
Python sorted() 函数
"""
a = [5,7,6,3,4,1,2]
b = sorted(a)
# Outputs
# [1, 2, 3, 4, 5, 6, 7]
 
L=[('b',2),('a',1),('c',3),('d',4)]
sorted(L, cmp=lambda x,y:cmp(x[1],y[1]))   # 利用cmp函数
# Outputs
# [('a', 1), ('b', 2), ('c', 3), ('d', 4)]
sorted(L, key=lambda x:x[1])               # 利用key
# Outputs
# [('a', 1), ('b', 2), ('c', 3), ('d', 4)]
 
students = [('john', 'A', 15), ('jane', 'B', 12), ('dave', 'B', 10)]
sorted(students, key=lambda s: s[2])            # 按年龄排序
# Outputs
# [('dave', 'B', 10), ('jane', 'B', 12), ('john', 'A', 15)]
sorted(students, key=lambda s: s[2], reverse=True)       # 按降序
# Outputs
# [('john', 'A', 15), ('jane', 'B', 12), ('dave', 'B', 10)]

```

#### 1.8 字典的运算

```python
# 字典的运算

"""
zip()，可以将两个数量相同的list一一对应组合。
通过取出dict中的key，values，将其颠倒组合，就能对value进行排序最大最小等操作。
注：Python3中的zip之后是一个对象而不是list，需要通过list()转成list。
"""
prices = {
    'ACME': 45.23,
    'AAPL': 612.78,
    'IBM': 205.55,
    'HPQ': 37.20,
    'FB': 10.75
}

# prices.values()可以取出value所有值作为list。为了对字典值执行计算操作，通常需要使用 zip() 函数先将键和值反转过来。
min_price = min(zip(prices.values(), prices.keys()))
# min_price is (10.75, 'FB')
max_price = max(zip(prices.values(), prices.keys()))
# max_price is (612.78, 'AAPL')

# 类似的，可以使用 zip() 和 sorted() 函数来排列字典数据：
prices_sorted = sorted(zip(prices.values(), prices.keys()))
# prices_sorted is [(10.75, 'FB'), (37.2, 'HPQ'),
#                   (45.23, 'ACME'), (205.55, 'IBM'),
#                   (612.78, 'AAPL')]

# 执行这些计算的时候，需要注意的是 zip() 函数创建的是一个只能访问一次的迭代器。 如果需要可以将zip后的list取出。
vtok = list(zip(prices.values(), prices.keys())))

```

#### 1.9 查找两字典的相同点

```python
# 查找两字典的相同点

""" 
字典中的key是可以进行集合操作的。而a.values不可以，需要转成set后操作。
"""
a = {'x': 1, 'y': 2, 'z': 3}
b = {'w': 10, 'x': 11, 'y': 2}

# Find keys in common
a.keys() & b.keys()  # { 'x', 'y' }
# Find keys in a that are not in b
a.keys() - b.keys()  # { 'z' }
# Find (key,value) pairs in common
a.items() & b.items()  # { ('y', 2) }

```

#### 1.10 删除序列相同元素并保持顺序

```python
# 删除序列相同元素并保持顺序

""" 
如果序列上的值为hashable类型，则可以通过集合与生成器解决这个问题。
其中集合是为了解决元素重复问题，yield则是依次返回元素到list。
"""
def dedupe(items):
    seen = set()
    for item in items:
        if item not in seen:
            yield item
            seen.add(item)


a = [1, 5, 2, 1, 9, 1, 5, 10]
list(dedupe(a))
# Outputs
# [1, 5, 2, 9, 10]

""" 
如果你想消除元素不可哈希（比如 dict 类型）的序列中重复元素的话，需改动代码，将序列元素转换成hashable类型，其实即是选取需要对比的value。
"""
def dedupe2(items, key=None):
    seen = set()
    for item in items:
        val = item if key is None else key(item)
        if val not in seen:
            yield item
            seen.add(val)

a = [{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 1, 'y': 2}, {'x': 2, 'y': 4}]
list(dedupe2(a, key=lambda d: (d['x'], d['y'])))
# Outputs
# [{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]
list(dedupe2(a, key=lambda d: d['x']))
# Outputs
# [{'x': 1, 'y': 2}, {'x': 2, 'y': 4}]

""" 
仅仅消除元素的话用set就足够。
对文件操作的话仅需进行如下变化，可以消除重复行：
with open(somefile,'r') as f:
for line in dedupe(f):
    ...
"""

```

---

> 参考：
>
> https://python3-cookbook.readthedocs.io/zh_CN/latest/
>
> https://github.com/yidao620c/python3-cookbook