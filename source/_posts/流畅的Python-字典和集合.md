---
title: 流畅的Python - 字典和集合 (三)
author: zranbo
abbrlink: 3dd06891
tags:
  - Fluent Python
categories: []
date: 2019-05-18 12:26:00
---
<b>泛映射类型</b> 抽象类：

在 collections.abc 模块中有 <b>[`Mpping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping)</b> 和 <b>[`MutableMapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.MutableMapping)</b> 为 `dict` 和其他一些类提供了最基本的接口
![](https://zranbo.oss-cn-beijing.aliyuncs.com/blog/dict-uml.png)

<b>可散列类型</b>
 - `str` `bytes` `frozenset`只能容纳可散列对象，`tuple` 只有当其中的所有元素都是可散列类型时，它才是可散列的。
 - 如果一个对象是可散列的，那么在它的生命周期内散列值不变。与此同时，该对象还需要实现 `__hash__()` 方法，并且如果需要和其他的对象散列值相比，就必须再实现 `__eq__()` (参见 <b>[`hashable`](https://docs.python.org/3/glossary.html#term-hashable)</b>)
 - 自定义类型的对象默认都是可散列的, (参见 <b>[`__hash__()`](https://docs.python.org/3/reference/datamodel.html#object.__hash__)</b>)。如果一个对象实现了 `__eq__()` 方法，并且在方法中用到了这个对象的内部状态的话，那么只有当所有这些内部状态都是不可变的情况下，这个对象才是可散列的。没有实现 `__eq__()` 方法时，自定义对象是可散列的，否则之后在保持 `__eq__()` 使用的内部状态不可变，才可散列。
 
 
eg.
```python
class TestObject(object):
    def __init__(self, n):
        self.n = n

    def __repr__(self):
        return str(self.n)


s = set()

t1 = TestObject(1)
t2 = TestObject(2)

s.add(t1)
s.add(t2)
print(s)
```
这里的结果是 `{2, 1}`

```python
class TestObject(object):
    def __init__(self, n):
        self.n = n

    def __repr__(self):
        return str(self.n)

    def __eq__(self, obj):
        return hash(self) == hash(obj)

s = set()

t1 = TestObject(1)
t2 = TestObject(2)

s.add(t1)
s.add(t2)
print(s)
```

我们只添加一个 `__eq__()` 的方法之后这段代码会报错，原因是在实现了 `__eq__()` 而没有实现`__hash__()` 的时候，默认 `__hash__()` 方法将设为 `None` 所以此时的错误为 `TypeError: unhashable type: 'TestObject'` 

```python
class TestObject(object):
    def __init__(self, n):
        self.n = n

    def __repr__(self):
        return str(self.n)

    def __eq__(self, obj):
        return hash(self) == hash(obj)

    def __hash__(self):
        return 1

s = set()

t1 = TestObject(1)
t2 = TestObject(2)

s.add(t1)
s.add(t2)
print(s)
```
而在实现了 `__hash__()` 方法之后，如果我们指定一个确定的值作为所有对象的 `hash` 的结果，这时候 `s` 就只有一个元素了，因为这个此时两个对象再进行比较的时候，通过 `__eq__()` 方法，但是此时两个对象的 `__hash__()` 都返回的是 `1` 因此就判定这两个对象是同一个，所以在 `set` 中只有一个了。

```python
class TestObject(object):
    def __init__(self, n):
        self.n = n

    def __repr__(self):
        return str(self.n)

    def __eq__(self, obj):
        return hash(self) == hash(obj)

    def __hash__(self):
        return id(self)

s = set()

t1 = TestObject(1)
t2 = TestObject(2)

s.add(t1)
s.add(t2)
print(s)
print(t1 is t2)
```

<b>字典的定义</b>

字典 `dict` 是一种映射类型的数据，我们可以用 `isinstance(obj, abc.Mapping)` 来判断任何一种对象是不是广义上的映射类型，这里的 `obj` 可以是内置的任何一种数据，当然也可是我们自己定义的任何一种数据。

<b>构造一个字典</b>

构造一个字典可以有很多种不同的形式
```python
d = dict(a=1, b=2, c=3)
print(d)
```
```python
d = {'a': 1, 'b': 2, 'c': 3}
print(d)
```
```python
d = dict([('a', 1), ('b', 2), ('c', 3)])
print(d)
```
```python
d = dict({'a': 1, 'b': 2, 'c': 3})
print(d)
```
```python
d = dict(zip(['a', 'b', 'c'], [1, 2, 3]))
print(d)
```
```python
data = [
    ('a', 1),
    ('b', 2),
    ('c', 3)
]
d = {k: v for k, v in data}
print(d)
```
<b>`collections.defaultdict` 和 `collections.OrderedDict`</b>

`collections` 是一个数据类型容器模块。
有关 [`defaultdict`](https://docs.python.org/3.7/library/collections.html#collections.defaultdict) 和 [`OrderedDict`](https://docs.python.org/3.7/library/collections.html#collections.OrderedDict) 的介绍和草看使用可以参阅文档。

<b>`dict` `defaultdict` 和 `OrderedDict` 对比</b>

| 方法名 | 说明 | dict | defaultdict | OrderedDict |
| :---: | :---: | :---: | :---: | :---: |
| `d.__contains__(k)` | `k in d` | ✓ | ✓ | ✓ |
| `d.__copy__()` | `copy.copy(d)` | ✗ | ✓ | ✗ |
| `d.__len__()` | `len(d)` | ✓ | ✓ | ✓ |
| `d.__setitem__(k, v)` | `d[k] = v` | ✓ | ✓ | ✓ |
| `d.__getitem__(k)` | `v = d[k]` | ✓ | ✓ | ✓ |
| `d.__delitem__(k)` | `del d[k]` | ✓ | ✓ | ✓ |
| `d.__iter__()` | `key`的迭代器 | ✓ | ✓ | ✓ |
| `d.keys()` | 获取所有`key` | ✓ | ✓ | ✓ |
| `d.values()` | 获取所有`value` | ✓ | ✓ | ✓ |
| `d.copy()` | 浅拷贝 | ✓ | ✓ | ✓ |
| `d.clear()` | 清空所有数据 | ✓ | ✓ | ✓ |
| `d.update(m, [**kargs])` | `m` 是映射或者键值对迭代器，更新 `d` 里对应的条目 | ✓ | ✓ | ✓ |
| `d.fromkeys(it, [initial])` | 迭代器 `it` 里的元素设置为映射里的 `key`，可选 `initial` 参数，作为这些键对应的值(默认 `None`) | ✓ | ✓ | ✓ |
| `d.setdefault(k, [default])` | 如果有键 `k`，把它的值设置为 `default` ，否则 `d[k] = default`，返回 `default` | ✓ | ✓ | ✓ |
| `d.get(k, [default])` | 返回 `k` 对应的值，如果没有 `k`，则返回 `default` | ✓ | ✓ | ✓ |
| `d.pop(k, [defaul]` | 返回 `k` 所应的值，弹出这个`k-v`。如果没有这个`k`，返回 `defaul` | ✓ | ✓ | ✓ |
| `d.items()` | 返回 `d` 里所有的 `k-v` | ✓ | ✓ | ✓ |
| `d.popitem()` | 随机弹出一个 `k-v` 对。会移除字典里最先插入的元素(FIFO)；可选 `last` 参数为`True`，弹出最后插入的元素(LIFO)。 | ✓ | ✓ | ✓ |
| `d.__missing__(k)` | 当 `__getitem__` 找不到对应 `k` 的时候会被调用 | ✗ | ✓ | ✗ |
| `d.default_factory` | 在 `__missing__` 函数中用以给未找到的元素设置值 | ✗ | ✓ | ✗ |
| `d.move_to_end(k, [last])` | 把 `k` 的元素移动到最前或最后(`last` 默认 `True`) | ✗ | ✗ | ✓ |
| `d.__reversed__()` | 返回倒序的 `key` 的迭代器 | ✗ | ✗ | ✓ |

<b>集合 `set` </b>

![](https://zranbo.oss-cn-beijing.aliyuncs.com/blog/set-uml.png)

集合是许多唯一对象的聚合。
```python
l = ['spam', 'spam', 'eggs', 'spam']
s = set(l)
print(list(s))
```

集合中的元素必须是可散列的，`set` 类型本身是不可散列的，但是 `frozenset` 可以。所以可以创建一个包含不同 `frozenset` 的 `set`。
