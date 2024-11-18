###	L2 Functions-en



####	Types of Expressions

Primitive expressions 原始表达式

Call expressions 调用表达式

评估调用表达式的规则



####	Environment Diagrams 环境图

环境图用来可视化解释器的过程，以便我们真正理解程序是如何执行的。

Frame：跟踪名称和值之间的绑定关系。

Global Frame：



####	Pure Functions & Non-Pure Functions

纯函数：只返回值。

非纯函数：有副作用。



####	交互模式下运行 Python3

```bash
python3 -i hello.py
```



####	函数中的DocString

def 语句定义函数时，紧跟着第一行可以是 DocString。除了为人描述这个函数做了什么，还可以展示一些使用例子：

```python
from opertor import floordiv, mod

def divide_exact(n, d):
    """Return the quotient and remainder of dividing N by D.
   	>>> q, r = divide_exact(2013, 10)
    >>> q
    201
    >>> r
    3
    """
    return floordiv(n, d), mod(n, d)
```

可以增加选项，来模拟该会话：

```bash
python3 -m doctest hello.py
```

如果一切正常，则无任何输出，可以传递 -v 选项查看更多信息：

```bash
python3 -m doctest -v hello.py
```

doctest 是通过在特定文件上调用文档测试模块来运行的。



####	判断语句

Python 中布尔假的值有：False，zero，empty string 和 None 





####	迭代 iteration



####	通告 Announcements

调用表达式不允许你跳过对调用表达式的部分进行评估，在函数被调用时，所有部分都会

被始终进行评估。



####	表达式控制

逻辑运算中的短路现象。



####	高阶函数

断言 assert：

```python
def area(r, shape_constant):
    assert r > 0, 'A length must be positive'
    return r * r * shape_constant
```

泛化：

