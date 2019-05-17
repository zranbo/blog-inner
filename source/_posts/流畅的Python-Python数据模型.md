---
title: 流畅的Python - Python数据模型 (一)
tags:
  - Fluent Python
categories: []
author: zranbo
abbrlink: 1bbbb445
date: 2019-05-14 14:03:00
---
<b>Magic Method (Dunder Method)</b>

{% note info %} Python 里面有很多特殊方法，我们管他叫魔术方法(magic method)。之所以特殊，是因为这些方法不同于一般的对象方法的调用方式 `Object.function`，比如对于获取字符串对象 `s = "Hello"` 的长度,不同于其他一些语言的 `s.length()` 或者类似的方法，Python采用 `len(s)` 这样的形式。 {% endnote %}


{% blockquote ——Fluent Python %}
首先明确一点，特殊方法的存在是为了被 Python 解释器调用的，你自己并不需要调用它们。也就是说没有 `my_object.__len__()` 这种写法，而应该使用 `len(my_object`)。在执行 `len(my_object)` 的时候，如果 `my_object` 是一个自定义类的对象，那么 Python 会自己去调用其中由你实现的 `__len__` 方法。
{% endblockquote %}

而 Python 里面这样的特殊方法还有很多，列一个表格说明一些。

| 类别 | 方法名 |
| :---: | :--- |
| 字符串/字节序列表示形式 | `__repr__`、`__str__`、`__format__`、`__bytes__` |
| 数值转换 | `__abs__`、`__bool__`、`__complex__`、`__int__`、`__float__`、`__hash__`、`__index__` |
| 集合模拟 | `__len__`、`__getitem__`、`__setitem__`、`__delitem__`、`__contains__` |
| 迭代枚举 | `__iter__`、`__reversed__`、`__next__` |
| 可调用模拟 | `__call__` |
| 上下文管理 | `__enter__`、`__exit__` |
| 实例创建和销毁 | `__new__`、`__init__`、`__del__` |
| 属性管理 | `__getattr__`、`__getattribute__`、`__setattr__`、`__delattr__`、`__dir__` |
| 属性描述符 | `__get__`、`__set__`、`__delete__` |
| 类相关服务| `__prepare__`、`__instancecheck__`、`__subclasscheck__` |

上面表格里面列举的并不是所有的，还有很多比如与运算相关的特殊方法 `__add__` 、 `__abs__` 等都没有列举。

<b>一些常用操作对应的魔术方法</b>

 - `for` 迭代操作
 
 如果一个对象可以通过 `for` 循环，需要实现 `__iter__()` 和 `__next__()` 方法，并且需要在 `__next__()` 中通过抛出 `StopIteration` 异常来控制 `for` 的终止行为:

```python
class tObject(object):
    def __init__(self, init=0, stop=10):
        self.init = init
        self.data = init
        self.stop = stop

    def __iter__(self):
        return self

    def __next__(self):
        if self.data >= self.stop:
            raise StopIteration
        else:
            self.data += 1
            return self.data
            
obj = tObject()
for i in obj:
    print(i)
```

 - `in` 操作
 
 用 `in` 判断对象关系的时候，需要实现 `__contains__() `，但是如果没有实现 `__contains__()` ，会进行一次迭代操作：
 
```python
class tObject(object):
    def __init__(self, init=0, stop=10):
        self.init = init
        self.data = init
        self.stop = stop

    def __iter__(self):
        return self

    def __next__(self):
        if self.data >= self.stop:
            raise StopIteration
        else:
            self.data += 1
            return self.data

    def __contains__(self, key):
        if self.init < key < self.stop:
            return True
        else:
            return False
            
obj = tObject()

print(5 in obj)
```
但是如果没有实现 `__contains__()` 这里这里就会首先进行一次迭代操作，然后去判断。

 - `print()`或`str()`
 
 `__repr__()`的目的应该是能准确表达出对象。
 
 `__str__()`的目的是能友好的可读的表达对象。
 
 
 - `[]` 取值&赋值&操作
 
 分别是 `__getitem__()` 和 `__setitem__()`
 
```python
class tObject(object):
    def __init__(self):
        self.data = dict()

    def __getitem__(self, key):
        return self.data.get(key, 'N/A')

    def __setitem__(self, key, val):
        self.data[key] = val


obj = tObject()
obj['1'] = 'a'
print(obj['1'])
```
`__getitem__()` 方法用于返回参数 `key` 所对应的值，这个 `key` 可以是索引，关键字或切片对象，并且支持负数索引；如果 `key` 不是以上两种类型，就会抛出 `TypeError` ；如果索引越界，会抛 `IndexError` ；如果定义的是映射类型，当 `key` 参数不是其对象的键值时，则会抛出 `KeyError`