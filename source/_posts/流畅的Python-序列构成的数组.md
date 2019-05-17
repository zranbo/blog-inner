title: 流畅的Python - 序列构成的数组 (二)
author: zranbo
tags:
  - Fluent Python
categories: []
date: 2019-05-15 17:01:00
---
<b>序列类型概念</b>

谈起序列，我们能说的就有很多了，比如我们很常用的`str` , `list` , `tuple` 等，Python用C语言实现了丰富的序列类型：

 - 容器序列（这些序列可以存放不同类型的数据）
 
 `list` `tuple` 等
 
 - 扁平序列（这些序列只能存放同一种类型数据）
 
 `str` `bytes` `bytearray` `memoryview` `array.array` 等
 
 这里需要注意一下点的是，对于容器序列，里面存放的是任意类型的对象的<b>引用</b>，而扁平序列是一串连续的内存空间，保存的是这些数据的值。
 
```python
val = [1]
ctn = [val]

print(ctn)
val.append(2)
print(ctn)
```

<b>列表推导式</b>
```python
string = "hello world"
lst = [x.upper() for x in string]
```
这是一个很简单的列表推导式，当然，如果我们需要一些复杂的东西，可以这样变化一下

```python
colors = ['red', 'blue', 'pink']
sizes = ['M', 'L', 'XL']

goods = [(color, size) for color in colors for size in sizes]
print(goods)
goods = [(color, size) for size in sizes for color in colors]
print(goods)
```
调换了一下后面迭代的先后顺序，可以看到结果有了变化。


<b>生成器表达式</b>

从使用上来说，和列表推导式没有太大的区别，只是将`[ ]`换成`( )`。
列表推导式最终得到的是一个列表对象，而生成器表达式是一个元组对象。

<b>元组不仅仅是不可变的列表</b>

元组的一个特殊之处是对象一旦被创建，就不能更改，包括值和值所在的位置

```python
t1 = ('a', 'b')
t2 = ('a', 'b')
t3 = ('b', 'a')

print(t1 is t2)
print(t1 is t3)
```

正因为这个，元组就变得有点意思了，因为除了具体的值，元组还可以比列表多记录一项内容：这些数据的位置也可以用来保存一定的信息。

<b>序列的拆包</b>

注意这里我们并没有用元组的拆包，那是因为并不是只有元组才能拆包，而是一般的，对于大多数Python的序列，我们都可以这要去做
```python
eg = ('a', 'b', 'c')
ele1, ele2, ele3 = eg
print(ele1, ele2, ele3)

egl = ['a', 'b', 'c']
ele1, ele2, ele3 = egl
print(ele1, ele2, ele3)

eg2 = {'a': 1, 'b': 2, 'c': 3}
ele1, ele2, ele3 = eg2
print(ele1, ele2, ele3)

eg3 = 'abc'
ele1, ele2, ele3 = eg3
print(ele1, ele2, ele3)
```
值得注意的一点是，对于字典 `dict` 的拆包，并不是把 `key-value` 这个键值对拿出来了，而只是把 `key` 拆开了，其实对于这个我们一点也不惊讶，毕竟如果我们需要对字典的 `key-value` 都进行迭代，并不是 `for data in dict` 而是 `for k, v in dict.items()` 注意观察到其实我们也发现了，这里不就是用了对字典的 `key-value` 拆包的方法么？

<b>序列拆包中 `*` 的作用</b>

对于拆包过程中，我们可以使用 `*` 来包含多个元素
```python
t = ('a', 'b', 'c', 'd', 'e', 'f')
ele1, ele2, *eles = t
print(ele1, ele2, eles)
```

而 `*` 表示的多个元素则会放到一个列表中去。

而且这个 `*` 可以放到中间表示出了首末若干元素的其他所有元素
```python
t = ('a', 'b', 'c', 'd', 'e', 'f')
ele1, ele2, *eles, ele3 = t
print(ele1, ele2, eles, ele3)
```

<b>元组和列表的区别</b>

这里的区别体现在一些内置的属性方法上。列表说明

| 属性方法 | 说明 | 列表 | 元组 |
| :---: | :---: | :---: | :---: |
| `s.__getnewargs__()` | `pickle` 中支持更加优化的序列化 | ✗ | ✓ |
| `s.__add__(s2)` | `s + s2`，拼接 | ✓ | ✓ |
| `s.__contains__(e)` | `s` 是否包含 `e` | ✓ | ✓ |
| `s.index(e)` | 在 `s` 中找到元素 `e` 第一次出现的位置 | ✓ | ✓ |
| `s.count(e)` | `e` 在 `s` 中出现的次数 | ✓ | ✓ |
| `s.__getitem__(p)` | 	`s[p]`，获取位置 `p` 的元素 | ✓ | ✓ |
| `s.__iter__()` | 获取 `s` 的迭代器 | ✓ | ✓ |
| `s.__len__()` | `len(s)`，元素的数量 | ✓ | ✓ |
| `s.__mul__(n)` | `s * n`，`n` 个 `s` 的重复拼接 | ✓ | ✓ |
| `s.__rmul__(n)` | `n * s`，反向拼接 * | ✓ | ✓ |
| `s.__delitem__(p)` | 把位于 `p` 的元素删除 | ✓ | ✗ |
| `s.__reversed__()` | 返回 `s` 的倒序迭代器 | ✓ | ✗ |
| `s.__setitem__(p, e)` | `s[p] = e`，把元素 `e` 放在位置 `p`，替代已经在那个位置的元素 | ✓ | ✗ |
| `s.__imul__(n)` | `s *= n`，就地重复拼接 | ✓ | ✗ |
| `s.__iadd__(s2)` | `s += s2`，就地拼接 | ✓ | ✗ |
| `s.insert(p, e)` | 在位置 `p` 之前插入元素`e` | ✓ | ✗ |
| `s.append(e)` | 在尾部添加一个新元素 | ✓ | ✗ |
| `s.clear()` | 删除所有元素 | ✓ | ✗ |
| `s.copy()` | 浅复制 | ✓ | ✗ |
| `s.extend(it)` | 把可迭代对象 `it` 追加给 `s` | ✓ | ✗ |
| `s.pop([p])` | 删除最后或者是（可选的）位于 `p` 的元素，并返回它的值 | ✓ | ✗ |
| `s.remove(e)` | 删除 `s` 中的第一次出现的 `e` | ✓ | ✗ |
| `s.reverse()` | 就地倒序 | ✓ | ✗ |
| `s.sort([key], [reverse])` | 就地对 `s` 中的元素进行排序，可选的参数有键（`key`）和是否倒序（`reverse`） | ✓ | ✗ |

