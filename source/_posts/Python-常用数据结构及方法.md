---
title: Python 常用数据结构(列表,字典,元组,集合,序列,栈,队列)及方法
date: 2021-10-17 11:30:15
updated:
categories: [Python]
tags: [Python,数据结构]
---
## 列表
`insert`、`remove`、`sort` 等方法只修改列表，不输出返回值——返回的默认值为 `None` 。这是所有 Python 可变数据结构的设计原则。

不是所有数据都可以排序或比较。例如，`[None, 'hello', 10]` 就不可排序，因为整数不能与字符串对比，而 None 不能与其他类型对比。
<!-- more -->
常用方法。

|                方法名                 | 说明                                                         |
| :-----------------------------------: | :----------------------------------------------------------- |
|            list.append(x)             | 在列表末尾添加一个元素，相当于 a[len(a):] = [x]              |
|         list.extend(iterable)         | 用可迭代对象的元素扩展列表。相当于 a[len(a):] = iterable     |
|           list.insert(i, x)           | 在指定位置插入元素。第一个参数 i 是插入元素的索引            |
|            list.remove(x)             | 从列表中删除第一个值为 x 的元素。未找到指定元素时，触发 ValueError 异常。 |
|             list.pop([i])             | 删除列表中指定位置的元素，并返回被删除的元素。未指定位置时，a.pop() 删除并返回列表的最后一个元素。(方法签名中 i 两边的方括号表示该参数是可选的，不是要求输入方括号。这种表示法常见于 Python 参考库） |
|             list.clear()              | 删除列表里的所有元素，相当于 del a[:] 。                     |
|     list.index(x[, start[, end]])     | 返回列表中第一个值为 x 的元素的零基索引。未找到指定元素时，触发 ValueError 异常。可选参数 start 和 end 是切片符号，用于将搜索限制为列表的特定子序列。返回的索引是相对于整个序列的开始计算的，而不是 start 参数。例如：list.index(3,0,2) |
|             list.count(x)             | 返回列表中元素 x 出现的次数。                                |
| list.sort(*, key=None, reverse=False) | 就地排序列表中的元素，关于自定义排序参数，详见 sorted()      |
|            list.reverse()             | 翻转列表中的元素。                                           |
|              list.copy()              | 返回列表的浅拷贝。相当于 a[:] 。浅拷贝：拷贝引用，不拷贝内存。 |



## 双端队列

collections.deque 是双端队列，可以在任意位置插入元素，可以从队头和队尾插入元素。

调用方法：`from collections import deque`

常用方法

| 方法名                    | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| append(x)                 | 添加 x 到右端。                                              |
| appendleft(x)             | 添加 x 到左端。                                              |
| clear()                   | 移除所有元素，使其长度为0.                                   |
| copy()                    | 创建一份浅拷贝。                                             |
| count(x)                  | 计算 deque 中元素等于 x 的个数。                             |
| extend(iterable)          | 扩展deque的右侧，通过添加iterable参数中的元素。              |
| extendleft(iterable)      | 扩展deque的左侧，通过添加iterable参数中的元素。注意，左添加时，在结果中iterable参数中的顺序将被反过来添加。 |
| index(x[, start[, stop]]) | 返回 x 在 deque 中的位置（在索引 start 之后，索引 stop 之前）。 返回第一个匹配项，如果未找到则引发 ValueError。 |
| insert(i, x)              | 在位置 i 插入 x 。                                           |
| pop()                     | 移去并且返回一个元素，deque 最右侧的那一个。 如果没有元素的话，就引发一个 IndexError。 |
| popleft()                 | 移去并且返回一个元素，deque 最左侧的那一个。 如果没有元素的话，就引发 IndexError。 |
| remove(value)             | 移除找到的第一个 value。 如果没有的话就引发 ValueError。     |
| reverse()                 | 将deque逆序排列。返回 None 。                                |
| rotate (*n=1*)            | 向右循环移动 *n* 步。 如果 *n* 是负数，就向左循环。          |
| maxlen                    | Deque的最大尺寸，如果没有限定的话就是 None 。只读属性，不能被修改 |

> iterable 表示什么？
>
> iterable 表示可迭代对象，意思是可由迭代器遍历的对象。
>
> ```shell
> >>> items = [ "one","two","three","four" ]
> >>> iterator = iter(items)
> >>> next(iterator) 
> 'one'
> >>> next(iterator) 
> 'two'
> >>> next(iterator) 
> 'three'
> >>> next(iterator) 
> 'four'
> ```
>
> 列表，字典，字符串等都属于可迭代对象（iterable）。
>
> iterator 表示迭代器。

除了以上操作，deque 还支持迭代、封存、len(d)、reversed(d)、copy.copy(d)、copy.deepcopy(d)、成员检测运算符 in 以及下标引用例如通过 d[0] 访问首个元素等。 索引访问在两端的复杂度均为 O(1) 但在中间则会低至 O(n)。 如需快速随机访问，请改用列表。



## 元组

Python的元组与列表类似，不同之处在于元组的元素不能修改。元组使用小括号，列表使用方括号。

元组创建很简单，只需要在括号中添加元素，并使用逗号隔开即可。

```python
tup = ()  		# 创建空元组
tup = (12,) 	# 创建只含一个元素的元组时，要加逗号
```

访问元组

```python
tup[0]       # 访问 0 号元素
tup[1,3]     # 访问 1 2 号元素
```

删除元组

```python
del tup
```

常用方法

| 方法名             | 说明                   |
| ------------------ | ---------------------- |
| cmp(tuple1,tuple2) | 比较两个元组元素       |
| len(tuple)         | 计算元组元素个数。     |
| max(tuple)         | 返回元组中元素最大值。 |
| min(tuple)         | 返回元组中元素最小值。 |
| tuple(list)        | 将列表转换为元组       |



## 队列

队列特点：先入先出

在双端队列作一些限制就可以作为队列使用。

初始化：`queue = deque(["Hello","World"])`

入队：`queue.append("Larry")`

出队：`queue.popleft() `

判断非空  len(queue)!=0



## 序列

序列和元组一样，都不可修改，但可以将多个序列连接为一个序列，也可以删除序列。

序列就是可包含多种数据类型的元组。

```shell
>>> t = 12345, 54321, 'hello!'
>>> t
(12345, 54321, 'hello!')
>>> u = t,'world';
>>> u
((12345, 54321, 'hello!'), 'world')      // 元组中可包含元组
```



## 堆栈

堆栈特点：先入后出

在双端队列作一些限制就可以作为堆栈使用。

初始化：`stack = deque(["Hello","World"])`

入栈：`stack.append("nice")`

出栈：`stack.pop()`       # 返回弹出的元素

判断非空：len(stack)!=0

## 集合

集合是由不重复元素组成的无序容器。基本用法包括成员检测、消除重复元素。集合对象支持合集、交集、差集、对称差分等数学运算。

创建集合用花括号或 set()函数。注意，创建**空集合**只能用 `set()`，不能用 `{}`，`{}` 创建的是空字典。

```
basket = {"abc","def","abc"}
print(basket)              # "abc","def" 会自动去除重复元素
a = set('abracadabra')
b = set('alacazam')
a                                 
a - b                              # 在 a 中 但不在 b 中的元素
a | b                              # 在 a 中或在 b 中的元素
a & b                              # 既在 a 中也在 b 中的元素
a ^ b                              # 在 a 中或在 b 中，但不能 a b 都有。
```



## 字典

字典以 关键字 为索引，关键字通常是字符串或数字，也可以是其他任意不可变类型。只包含字符串、数字、元组的元组，也可以用作关键字，列表不能当关键字，因为列表可以用索引、切片、`append()` 、`extend()` 等方法修改。花括号 `{}` 用于创建空字典。另一种初始化字典的方式是，在花括号里输入逗号分隔的键值对，这也是字典的输出方式。

```
dic = {"jack":18,"larry":22,"mac":25}        # 创建字典
dic["jack"] = 23.                            # 修改键所对应的值
del dic['mac']															 # 删除元素
list(dic)																		 # 将字典的键作为列表返回
sorted(dic)																	 # 对键进行排序
```

dict() 构造函数可以直接用键值对序列创建字典：

```python
dict([('larry', 1314), ('sky', 23), ('jack', 18)]) # {'larry': 1314, 'sky': 23, 'jack': 18}
dict(mac=99,windows=99)      #  {'mac': 99, 'windows': 99}
```

字典推导式可以用任意键值表达式创建字典：

```python
{x: x**2 for x in (2, 4, 6)}
{2: 4, 4: 16, 6: 36}
```

遍历遍历

用 `items()` 方法可同时取出键和对应的值：

```python
dic={"apple":100,"windows":24}
for k, v in dic.items():
     print(k,v)
```

在序列中循环时，用 enumerate()  函数可以同时取出位置索引和对应的值：

```python
fruit={"apple":100,"banana":26}
for i,v in enumerate(fruit):
		print(i,v)
```





## 列表推导式

使用 列表推导式子，可以方便地构造特定列表。

使用以下代码创建平方值列表：

```python
squares = []
for x in range(10):
		squares.append(x**2)
```

注意，这段代码创建（或覆盖）变量 `x`，该变量在循环结束后仍然存在。下述方法可以无副作用地计算平方列表：

### map

```python
squares = list(map(lambda x: x**2,range(10)))
```

> map 第一个参数 function 以参数序列中的每一个元素调用 function 函数，返回包含每次 function 函数返回值的新列表。
>
> map(function, iterable, ...)，可以提供多个列表，函数的参数个数和列表个数相同。
>
> ```python
> >> list(map(lambda x: x ** 2, [1, 2, 3, 4, 5]))   # 使用 lambda 匿名函数
> [1, 4, 9, 16, 25]
> ```
>
> ```python
> >> map(lambda x, y: x + y, [1, 3, 5, 7, 9], [2, 4, 6, 8, 10])
> [3, 7, 11, 15, 19]
> ```
>
> 注：map 函数 Python 2.x 返回 列表，  Python 3.x 返回迭代器



### 表达式+for子句

列表推导式的方括号内包含以下内容：一个表达式，后面为一个 `for` 子句，然后，是零个或多个 `for` 或 `if` 子句。结果是由表达式依据 `for` 和 `if` 子句求值计算而得出一个新列表。

```python
>>> [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y  #(x,y)表示元组，其值不可修改
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
```



## 嵌套的列表推导式

构造一个：3x4 矩阵

```
matrix = [
     [1, 2, 3, 4],
     [5, 6, 7, 8],
     [9, 10, 11, 12],
]
```

常规方法如下：

```python
>>> transposed = []
>>> for i in range(4):
...     transposed.append([row[i] for row in matrix])
```

还可以使用列表推导式：

```python
[[row[i] for row in matrix] for i in range(4)]
```



## 元素删除(`del` 语句)

del 语句按索引，而不是值从列表中移除元素。与返回值的 pop() 方法不同， del 语句也可以从列表中移除切片，或清空整个列表（之前是将空列表赋值给切片）。 例如：

```
a = [1,2,3,4,5]
del a[0]       # 删除第一个元素
del a[2:4]     # 删除 2号 和 3号 元素
del a          # 删除整个变量
```



## 注意

对不同类型的对象来说，只要待比较的对象提供了合适的比较方法，就可以使用 `<` 和 `>` 进行比较。

