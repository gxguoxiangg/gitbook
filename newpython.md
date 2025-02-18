####	zip() 内置函数

**zip(*iterables, strict=False)**

zip 返回元组的迭代器，其中第 i 个元组包含的是每个参数迭代器的第 i 个元素。



####	Unpacking 解包 

拆解可迭代对象（如列表、元组、字典），并将其元素分别赋值给变量。

解包的用法示例：

1.传递多个参数

```python
def add(x, y, z):
    return x + y + z
nums = [1, 2, 3]
print(arr(*nums))	# 等效于 add(1, 2, 3)
```

`*nums` 把列表元素展开后作为单独参数传入函数。

2.用户多余变量

```python
a, *b, c = [1, 2, 3, 4, 5]
print(a) # 1
print(b) # [2, 3, 4]
print(c) # 5
```

`*b` 取中间所有元素，剩下的 `a` 和 `c` 分别取两端。

| **用法**         | **示例**               | **作用**              |
| ---------------- | ---------------------- | --------------------- |
| **基本解包**     | `a, b = (1, 2)`        | 解开元组或列表        |
| **解包传参**     | `func(*args)`          | 传递列表元素作为参数  |
| **结合 `zip()`** | `zip(*pairs)`          | 反向解包 `zip()` 结果 |
| **用于剩余元素** | `a, *b, c = [1,2,3,4]` | `*b` 获取中间元素     |



####	iter() 

iter() 将可迭代对象 iterable 转换为迭代器 `iterator = iter(iterable)`

for 循环本质就是不断调用 `iter()` 和 `next()`，直到 `StopIteration` 结束：

```python
arr = [1, 2, 3]
it = iter(arr)
while True:
    try:
        print(next(it))
    except StopIteration:
        break
```

 等效于：

```python
for x in arr:
    print(x)
```

进阶用法：

1.iter(callable, sentinel)

创建基于函数的迭代器：

- `callable`：一个不带参数的可调用对象
- `sentinel`：哨兵值，当返回这个值时，停止迭代

不断读取用户输入，直到输入 "exit"

```python
user_input = iter(input, "exit")
for line in user_input
	print(f"You typed: {line}")
# 每次迭代 iter(input, "exit") 会调用 input, 直到用户输入 exit 终止。
```



2.iter()自定义迭代器

python允许创建自己的迭代器，必须实现：

- `__iter__()`：返回迭代器对象本身（self）
- `__next__()`：返回下一个元素，当没有元素时抛出 `StopIteration`



3.iter() 在流式处理中的应用

逐行读取大文件：

```python
with open("large_file.txt") as f:
    for line in iter(f.readline, "")
    	print(line.strip())
# 相比 f.readlines, 这种方式不会一次加载整个文件，适合处理超大文件
```



4.iter() 结合 map() 进行惰性计算

计算平方，但不立即执行：

```python
arr = [1, 2, 3, 4]
squr = map(lambda x: x ** 2, arr)
print(next(squr)) # 1
print(next(squr)) # 4
# map() 返回的是迭代器，计算只在 next() 被调用时发生
```

| 用法                               | 说明                                     |
| ---------------------------------- | ---------------------------------------- |
| `iter(iterable)`                   | 将可迭代对象转换成迭代器                 |
| `next(iterator)`                   | 获取下一个元素                           |
| `iter(callable, sentinel)`         | 直到 `callable()` 返回 `sentinel` 时终止 |
| 自定义 `__iter__()` & `__next__()` | 让对象支持迭代                           |
| 结合 `zip()`、`map()`、`open()`    | 用于流式处理 & 惰性计算                  |



####	yield 关键字

yield 是 python 里用于创建生成器（generator）的关键字，它让函数变成可迭代的，支持惰性求值的对象，适合处理大数据流、节省内存、控制流等场景。

return：终止函数并返回一个值。

yield：暂停函数执行，并返回一个值，保留函数状态，下一次调用时从暂停点继续。

**生成器本质上是特殊的迭代器，自动实现 `__iter()__` 和 `__next()__`。**





####	set、frozenset 内置集合

set 是一种**无序多项集**，常用于成员检测、去除重复项、以及数学中的集合运算。

**set 类型是可变的** -- 其内容可以使用 add() 和 remove()  这样的方法来改变。由于是可变类型，它没有哈希值，且不能被用作字典的键或其他集合的元素。**frozenset 类型是不可变**并且为 hashable -- 其内容在被创建后不能再改变；因此它可以被用作字典的建或其他集合的元素。

集合可用多种方式来创建：

- 使用花括号内逗号分隔元素的方式：`{jack, tom}`
- 使用集合推导式：`{c for c in 'abcdef' if c not in 'abc'}`
- 使用类构造器：`set(), set('foo'), set(['a', 'b', 'abc'])`

常用方法：

- isdisjoint(other)

  两个集合没有交集时，返回 True。

- set <= other、set < other

  前者判断是否为子集，后者判断是否为真子集。

- set >= other、set > other

  超集判断。

- set | other | ...

  返回它们的并集。

- set & other & ...

  返回它们的交集。

- set - other - ...

  返回它们的差集。

- set ^ other

  返回它们的对称差集，即属于 set 或 other ，但不能同时属于两者。

- set.copy()

  返回集合的浅拷贝。

####	装饰器

将一个函数转换为另一个函数，可以简单的放一个注解，这称为装饰器。

```python
def fa():
    ....
fa = transform(fa)

# 等价于
@transform
def fa():
    .....
```



####	浅拷贝和深拷贝

**浅拷贝（Shallow Copy）** 指的是 **创建一个新对象，但只复制原对象的“引用”**，而**不会复制嵌套的子对象**。这意味着：

- **新对象和原对象共享子对象（即嵌套的可变对象，仍然指向相同的内存地址）。**
- **修改新对象的嵌套可变对象时，原对象也会受到影响**（因为它们指向同一块内存）。
- **但新对象本身是独立的**，不影响原对象的顶层结构。

python 提供 `copy` 模块来进行浅拷贝和深拷贝。

| **拷贝方式**      | **特点**                 | **是否共享子对象**     | **适用场景**           |
| ----------------- | ------------------------ | ---------------------- | ---------------------- |
| `=` 赋值          | 直接复制引用             | **是**（完全一样）     | 仅适用于完全相同的对象 |
| `copy.copy()`     | 浅拷贝，只复制顶层对象   | **是**（嵌套对象共享） | 适用于浅层结构的对象   |
| `copy.deepcopy()` | 深拷贝，递归复制所有对象 | **否**（完全独立）     | 适用于复杂嵌套结构     |



####	函数的特殊参数

| **参数类型**                      | **语法**         | **特点**                     |
| --------------------------------- | ---------------- | ---------------------------- |
| **位置参数**                      | `func(a, b)`     | 按顺序传递                   |
| **关键字参数**                    | `func(a=1, b=2)` | 通过参数名指定               |
| **默认参数**                      | `func(a=1)`      | 未传递时使用默认值           |
| **`\*args`（可变位置参数）**      | `func(*args)`    | 任意数量的位置参数（元组）   |
| **`\**kwargs`（可变关键字参数）** | `func(**kwargs)` | 任意数量的关键字参数（字典） |
| **仅限位置参数**                  | `func(a, /)`     | 不能用关键字传递             |
| **仅限关键字参数**                | `func(*, a)`     | 只能用关键字传递             |



####	内置函数

all(iterable)、any(iterable)

如果 iterable 的所有元素均为真值（或迭代对象为空），返回 True：

```python
def all(iterable):
    for element in iterable:
        if not element:
            return False
    return True
```

any 则是 iterable 中的任一元素为真值，就返回 True。

常用方式，如判断序列中是否有想要的值，或是否全为某一值：

```python
flag = any(x == 2 for x in arr)
```



###	内置类型

####	int类型

1. int.bit_count()：

   返回整数的绝对值的二进制表示中 1 的个数。

####	bool类型

1. False 和 True 需要使用 int() 执行显式转换才能使其变为0，1。

####	iterator类型

1. 迭代器对象自身需要支持以下两个方法，它们共同组成了迭代器协议： ``__iter__()`` 和`__next__()`
2. generator 生成器类型：使用 yield 关键字实现迭代器协议的简易方式。



####	list, tuple, range序列类型

**通用序列操作：**

- 拼接：s + t 、s * n
- 切片：s[i:j:k]
- 最值项：min(s)、max(s)
- 索引号：s.index(x,i[, j])
- 计数：s.count(x)

1. 相同类型的序列可以比较，使用的是**字典序**比较。
2. `in` 和 `not in` 操作在通常情况下仅被用于简单的成员检测，但在 str，bytes 中可以使用它们来进行子序列检测。
3. 拼接不可变序列总是会产生新的对象，这意味着重复拼接来构建序列的运行时开销将会基于序列总长度的平方。如果要获得线性的运行时开销，可以改用以下替代方案：
   - str：可以构建列表，并在最后使用 str.join(iterable)
   - bytes：bytes.join()
   - tuple：改为扩展 list 类
4. 使用 index 方法，两个额外参数大致相当于使用 s[i:j].index(x)，但是不会复制任何数据，并且返回的索引是相对于原序列的。



**可变序列类型：**

- `s[i:j] = t`：将 s 从 i 到 j 的切片替换为可迭代对象 t 的内容。
- `del s[i:j]`：等同于 s[i:j] = []。
- `s.clear()`：从 s 中 移除所有项。
- `s.copy()`：创建 s 的浅拷贝（等同于 s[:]）。
- `s.extend(t) or s += t`：用 t 的内容扩展 s。
- `s.pop() or s.pop(i)`：提取在 i 位置的项，并将其删除；默认参数 i 为 -1，即会提取删除最后一项。
- `s.remove(x)`：从 s 中移除第一个 s[i] 等于 x 的条目。
- `s.reverse()`：原地反转。

**列表list：**

可变序列

- sort(*, key=None, reverse=False)：

  原地稳定排序方法，默认使用 `<` 来进行各项之间的比较。

  接收两个仅限关键字形式传入的参数

**元组tuple：**

不可变序列，**决定生成元组的其实是逗号而不是圆括号，圆括号只是可选的**。

**range对象：**

便是不可变的数字序列，通常在 for循环中使用。range对象实现了一般序列的所有操作，除了拼接和重复除外。



####	文本序列类型 str

str 提供了大量的方法：

- str.endswith(*suffix*, [, *start*[, *end*]]) :

  ​	如果字符串以指定的 suffix 结束返回 True，否则返回 False。如果有可选项 start，将从所指定位置开始查找。

  ​	类似还有：str.startswith

- str.find(*sub*, [, *start*[, end]])

- str.isalnum()

- str.isalpha()

- str.isdigit()

- str.islower()

- str.join(*iterable*)

- str.lstrip([*chars*])：

  ​	返回原字符串的副本，移除其中的前导字符。*chars* 参数为指定要移除字符的字符串。如果省略或为 `None`, 则 *chars* 参数默认移除空白符。

- str.removeprefix()：

  ​	删除单个前缀字符串，而不是全部给定集合中的字符。

- str.partition(*sep*)：

  ​	在 *sep* 首次出现的位置拆分字符串，返回一个 3 元组，（分隔符之前、分隔符本身、分隔符之后），如果分隔符未找到，则返回的 3 元组中包含字符本身以及两个空字符串。

  ​	类似还有：str.rpartition

- str.replace(*old*, *new*, *count=-1*)：

  ​	返回字符串的副本，其中出现的所有字符串 *old* 都将被替换为 *new*。如果给出了 *count*,则只替换前 *count* 次出现。如果 *count* 未指定或为 `-1`，则全为替换。

-  str.rfind(*sub*[, *start*[, *end*]])：

  ​	返回子字符串 *sub* 在字符串内被找到的最大（最右）索引，未找到则返回 `-1`

- str.**split**(*sep=None, maxsplit=-1*)：

  ​	返回一个由字符串内单词组成的列表，使用 *sep* 作为分隔字符串。如果给出了 *maxsplit*，则最多进行 *maxsplit* 次拆分。

  ​	如果给出了 *sep*，则连续的分隔符不会被组合在一起而是会被视为分隔空字符串 (例如 `'1,,2'.split(',')` 将返回 `['1', '', '2']`)。 *sep* 参数可能是由多个字符组成的单个分隔符 (要使用多个分隔符进行拆分，请使用 [`re.split()`](https://docs.python.org/zh-cn/3/library/re.html#re.split))。 使用指定的分隔符拆分一个空字符串将返回 `['']`。

- str.rstrip([*chars*]):

  ​	返回原字符串的副本，移除其中的末尾字符。如果 *chars* 省略，则默认移除空白字符。

- 



####	二进制序列类型 --- bytes，bytearray，memoryview

操作二进制数据的核心内置类型是 [`bytes`](https://docs.python.org/zh-cn/3/library/stdtypes.html#bytes) 和 [`bytearray`](https://docs.python.org/zh-cn/3/library/stdtypes.html#bytearray)。 它们由 [`memoryview`](https://docs.python.org/zh-cn/3/library/stdtypes.html#memoryview) 提供支持，该对象使用 [缓冲区协议](https://docs.python.org/zh-cn/3/c-api/buffer.html#bufferobjects) 来访问其他二进制对象所在内存，不需要创建对象的副本。

**bytes 对象**



####	映射类型 --- dict

字典的构建方式：

- 字典推导式：`{x: x ** 2 for x in range(10)}`
- 使用类型构造器: `dict()`, `dict([('foo', 100), ('bar', 200)])`, `dict(foo=100, bar=200)`

成员方法：

- list(d)

  ​	返回字典 *d* 中使用的所有键的列表。

- del d[key]

  ​	将 `d[key]` 从 *d* 中移除。

-  iter(d)

  ​	返回以字典的键为元素的迭代器。

- get(*key, default=None*)

  ​	如果 *key* 存在于字典中则返回 *key* 的值，否则返回 *default*。

- keys()

  ​	返回由字典键组成的一个新视图。

- items()

  ​	返回由字典 (键，值) 组成的一个新视图。

- pop(*key*[, *default*])

  ​	如果 *key* 存在于字典中则将其移除并返回其值，否则返回 *default*。

- d | other

  ​	合并 *d* 和 *other* 中的键和值来创建一个新的字典，两者必须都是字典，当 *d* 和 *other* 有相同键时，*other* 的值优先。

- d |= other

  ​	用 other 的键和值更新字典 *d*



**字典视图对象**

由 [`dict.keys()`](https://docs.python.org/zh-cn/3/library/stdtypes.html#dict.keys), [`dict.values()`](https://docs.python.org/zh-cn/3/library/stdtypes.html#dict.values) 和 [`dict.items()`](https://docs.python.org/zh-cn/3/library/stdtypes.html#dict.items) 所返回的对象是 *视图对象*。 该对象提供字典条目的一个动态视图，这意味着当字典改变时，视图也会相应改变。

字典视图可以被迭代以产生与其对应的数据，并支持成员检测。



###	functools库

**reduce(function, iterable)**

将两个参数的 function 从左至右累积应用到 iterable 的条目，以便将该可迭代对象缩减为单个值。

如 reduce(lambda x, y: x+y, [1, 2, 3, 4, 5]) == ((((1+2)+3)+4)+5)







###	itertools库

> itertools 库：为高效循环创建迭代的函数库

无穷迭代器：

| 迭代器                                                       | 实参            | 结果                                  | 示例                                  |
| :----------------------------------------------------------- | :-------------- | :------------------------------------ | :------------------------------------ |
| [`count()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.count) | [start[, step]] | start, start+step, start+2*step, ...  | `count(10) → 10 11 12 13 14 ...`      |
| [`cycle()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.cycle) | p               | p0, p1, ... plast, p0, p1, ...        | `cycle('ABCD') → A B C D A B C D ...` |
| [`repeat()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.repeat) | elem [,n]       | elem, elem, elem, ... 重复无限次或n次 | `repeat(10, 3) → 10 10 10`            |

根据最短输入序列长度停止的迭代器：

| 迭代器                                                       | 实参                        | 结果                              | 示例                                                     |
| :----------------------------------------------------------- | :-------------------------- | :-------------------------------- | :------------------------------------------------------- |
| [`accumulate()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.accumulate) | p [,func]                   | p0, p0+p1, p0+p1+p2, ...          | `accumulate([1,2,3,4,5]) → 1 3 6 10 15`                  |
| [`chain()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.chain) | p, q, ...                   | p0, p1, ... plast, q0, q1, ...    | `chain('ABC', 'DEF') → A B C D E F`                      |
| [`chain.from_iterable()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.chain.from_iterable) | iterable                    | p0, p1, ... plast, q0, q1, ...    | `chain.from_iterable(['ABC', 'DEF']) → A B C D E F`      |
| [`filterfalse()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.filterfalse) | predicate, seq              | predicate(elem) 未通过的 seq 元素 | `filterfalse(lambda x: x<5, [1,4,6,3,8]) → 6 8`          |
| [`groupby()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.groupby) | iterable[, key]             | 根据key(v)值分组的迭代器          | `groupby(['A','B','DEF'], len) → (1, A B) (3, DEF)`      |
| [`islice()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.islice) | seq, [start,] stop [, step] | seq[start:stop:step]中的元素      | `islice('ABCDEFG', 2, None) → C D E F G`                 |
| [`pairwise()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.pairwise) | iterable -- 可迭代对象      | (p[0], p[1]), (p[1], p[2])        | `pairwise('ABCDEFG') → AB BC CD DE EF FG`                |
| [`starmap()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.starmap) | func, seq                   | func(*seq[0]), func(*seq[1]), ... | `starmap(pow, [(2,5), (3,2), (10,3)]) → 32 9 1000`       |
| [`zip_longest()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.zip_longest) | p, q, ...                   | (p[0], q[0]), (p[1], q[1]), ...   | `zip_longest('ABCD', 'xy', fillvalue='-') → Ax By C- D-` |

排列组合迭代器：

| 迭代器                                                       | 实参                 | 结果                                  |
| :----------------------------------------------------------- | :------------------- | :------------------------------------ |
| [`product()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.product) | p, q, ... [repeat=1] | 笛卡尔积，相当于嵌套的for循环         |
| [`permutations()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.permutations) | p[, r]               | 长度r元组，所有可能的排列，无重复元素 |
| [`combinations()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.combinations) | p, r                 | 长度r元组，有序，无重复元素           |
| [`combinations_with_replacement()`](https://docs.python.org/zh-cn/3/library/itertools.html#itertools.combinations_with_replacement) | p, r                 | 长度r元组，有序，元素可重复           |

| 例子                                       | 结果                                              |
| :----------------------------------------- | :------------------------------------------------ |
| `product('ABCD', repeat=2)`                | `AA AB AC AD BA BB BC BD CA CB CC CD DA DB DC DD` |
| `permutations('ABCD', 2)`                  | `AB AC AD BA BC BD CA CB CD DA DB DC`             |
| `combinations('ABCD', 2)`                  | `AB AC AD BC BD CD`                               |
| `combinations_with_replacement('ABCD', 2)` | `AA AB AC AD BB BC BD CC CD DD`                   |
