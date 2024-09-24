---
title: python的文档注释（Docstring）
author: mmy83
date: 2024-09-24 17:36:00 +0800
categories: [编程, python]
tags: [编程, python, 注释, 文档注释, docstring, "规范"]
math: true
mermaid: true
image:
  path: /images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-00.png
  lqip: data:image/webp;base64,UklGRkgAAABXRUJQVlA4IDwAAADQAQCdASoIAAQAAUAmJQBOiP/wMcRpwAD+1y6nfe22+GMgdbTd4kT1Lk4VOcbg0V5ALDQNl9KvtlPgAAA=
  alt: python的文档注释（Docstring）
---

## 介绍

&emsp;&emsp;注释一直是编程中不可或缺的一部分，它可以帮助我们理解代码的意图和逻辑。在 Python 中，注释通常用于解释代码的作用、参数和返回值等细节。python中的注释有两种类型：

+ 普通注释：以 # 开头的单行注释，这种注释在编译后会被忽略，不会在运行时执行。

+ 文档注释（DocStrings）：以 """ 开头，以 """ 结尾的多行注释。这种注释被称为文档注释，它会在代码运行时被解释器读取并显示。所以这种注释既可以给源码的开发者看，也可以给使用者看，编译后就是文档。

&emsp;&emsp;文档注释（DocStrings）可以这样理解，可以把它认为是注释，也可以认为是一个字符串，常见的字符串是为了赋值给一个变量，而这个字符串却不会赋值给任何变量。但它终究还是字符串，所有运行时还是存在的。而且python将指定位置的这种没有赋值给变量的字符串统一赋值给__doc__属性，而其他位置的因为既没有赋给变量或对象，也不在__doc__属性中，也就无法使用了。

```python

def print_msg(msg):
    '''
    这里是 DocStrings 注释
    
    Args:
        msg (str): the msg to show
    '''
    # 这里是普通注释
    print("hello" + msg)

```

## 规范（PEP）

&emsp;&emsp;docstring 是一种出现在模块、函数、类或者方法定义的第一行说明的字符串，这种 docstring 就会作为该对象的 __docs__ 属性。

&emsp;&emsp;从规范角度来说，所有的模块都应该有 docstrings，那些从模块中引入的函数和类也要有 docstrings。公有方法（包括 __init__ 构造器）也要有docstring。对于一个包来说，包的说明文档可能就来自__init__文件中的docstring.

```python
import collections
help(collections)
# 可以在 collections->__init__.py 中找到 help 函数返回的内容
```

## 格式

### 单行

```python
def kos_root():
    """Return the pathname of the KOS root directory."""
    global _kos_root
    if _kos_root: return _kos_root
    ...
```

> 注意
>
> + 即使字符串适合一行，也使用三引号，这样以后扩展起来比较容易
>
> + 结束引号与开始引号在同一行，这样单行看起来好看
>
> + 在文档字符串之前或之后都没有空行。
>
> + 文档字符串是以句号结尾的短语，描述的内容应该是"Do this", "Return that"，这个函数是用来干嘛的，函数参数如何，函数返回什么，不应该写其他无关内容。

### 多行

+ 多行 docstrings 由一个总结/摘要行（类似单行说明文档）、一个空行，然后是其他更详细的内容描述。

  + 总结/摘要行可以被自动索引工具使用，因此它需要符合位于第一行且后面空一行以和其他部分隔开这样的规范。

  + 总结/摘要行可以和开始的引号在一行，也可以在下一行。

  + 整个文档的缩进和第一行一样，如例子。

+ 一个类的所有 docstrings 后面，要插入一个空行。（和后面的代码隔开）

  + 一般类方法会用一个空行和其他内容分开，docstrings 和第一个方法之间要有个空行

+ 脚本（一个独立程序，比如一个.py文件）的 docstring 应该可以作为它的 usage 信息，比如脚本被不正确的调用，或者还使用 -h 参数时，就会打印出这个信息。

  + 这样的 docstring 应该记录了脚本的函数功能，还有命令行语法，环境变量以及文件等等。用法消息可能会很多，甚至好几个屏幕，但是起码要写的能让新用户可以正确使用这个脚本，同时对于熟练用户来说也是一个所有功能列表的快速参考。

+ 不要使用 Emacs 约定在运行文本中以大写形式提及函数或方法的参数。 Python 区分大小写，参数名称可用于关键字参数，因此文档字符串应记录正确的参数名称。最好在单独的行中列出每个参数。

```python
def complex(real=0.0, imag=0.0):
    """Form a complex number.

    Keyword arguments:
    real -- the real part (default 0.0)
    imag -- the imaginary part (default 0.0)
    """
    if imag == 0.0 and real == 0.0:
        return complex_zero
    ...
```

```python
class Variable:
    """
    Attributes:
    history (:class:`History` or None) : the Function calls that created this variable or None if constant
    derivative (variable type): the derivative with respect to this variable
    grad (variable type) : alias for derivative, used for tensors
    name (string) : a globally unique name of the variable
    """

    def __init__(self, history, name=None):
        global variable_count
        assert history is None or isinstance(history, History), history

        self.history = history
        self._derivative = None

        # This is a bit simplistic, but make things easier.
        variable_count += 1
        self.unique_id = "Variable" + str(variable_count)

        # For debugging can have a name.
        if name is not None:
            self.name = name
        else:
            self.name = self.unique_id
        self.used = 0

    def requires_grad_(self, val):
        """
        Set the requires_grad flag to `val` on variable.

        Ensures that operations on this variable will trigger
        backpropagation.

        Args:
            val (bool): whether to require grad
        """
        self.history = History()
```

## 风格

&emsp;&emsp;常见的 python 程序注释风格有 3 种：

+ google 推荐的注释风格，支持模块、类、函数、变量注释，非常详细，且易于阅读。

+ numpy 风格的注释，支持模块模块、类、函数、变量注释，同样易于阅读。

+ sphinx 风格的注释，支持函数、类，通过sphinx格式化输出比较漂亮，但阅读性弱一些。

### google 风格

#### 模块

&emsp;&emsp;如果是一个模块，那么文件的最开始，应该有类似以下内容：

```python
"""A one line summary of the module or program, terminated by a period.

Leave one blank line.  The rest of this docstring should contain an
overall description of the module or program.  Optionally, it may also
contain a brief description of exported classes and functions and/or usage
examples.

  Typical usage example:

  foo = ClassFoo()
  bar = foo.FunctionBar()
"""
```

#### 函数和方法

+ 文档字符串应该提供足够的信息来编写对函数的调用，而无需阅读函数的代码。文档字符串应描述函数的调用语法及其语义，但通常不描述其实现细节，除非这些细节与函数的使用方式相关

+ 函数的描述信息中有几个部分是比较特殊，需要进行特殊指明的。比如：

  + 参数，按名称列出每个参数，名称后面就跟着描述，用冒号/空格/换行符分开名称和描述。如果描述信息太长，单行超过 80 个字符，可以使用比参数名称多 2 或 4 个空格的悬空缩进（与文件中其余的 docstring 保持一致）。如果代码不包含相应的类型注释，则说明应包括所需的类型。如果函数接受 foo（变长参数列表）和/或 bar（任意关键字参数），则它们应列为 foo 和 bar。

  + 类似的还有 return 和 raise，这两种特殊类型的说明

&emsp;&emsp;如果是函数和方法，那么其 docstring 应该类似下面：

```python
def fetch_smalltable_rows(table_handle: smalltable.Table,
                          keys: Sequence[Union[bytes, str]],
                          require_all_keys: bool = False,
) -> Mapping[bytes, Tuple[str]]:
    """Fetches rows from a Smalltable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by table_handle.  String keys will be UTF-8 encoded.

    Args:
        table_handle: An open smalltable.Table instance.
        keys: A sequence of strings representing the key of each table
          row to fetch.  String keys will be UTF-8 encoded.
        require_all_keys: If True only rows with values set for all keys will be
          returned.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {b'Serak': ('Rigel VII', 'Preparer'),
         b'Zim': ('Irk', 'Invader'),
         b'Lrrr': ('Omicron Persei 8', 'Emperor')}

        Returned keys are always bytes.  If a key from the keys argument is
        missing from the dictionary, then that row was not found in the
        table (and require_all_keys must have been False).

    Raises:
        IOError: An error occurred accessing the smalltable.
    """
```

#### 类

```python
class SampleClass:
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam: bool = False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```

## 工具 & 插件

&emsp;&emsp;pyCharm 中使用两种方式进行方法的注释：Docstring format 和 Live Templates。

### Docstring format 添加方法注释

&emsp;&emsp;Docstring format 可通过下方路径进行设置，包括五种风格：Plain、Epytext、reStructuredText、Numpy、Google。

> File → to→ Settings → to→ Tools → to→ Python Integrated Tools → to→ Docstrings → to→ Docstring format

![Docstring format 添加方法注释](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-01.png)

&emsp;&emsp;使用方式为，在方法名下方输入三个双（单）引号，回车，自动生成。五种风格的样式如下：

```python
def docstrings_func_plain(parm_a, parm_b, parm_c):
    """
    Plain 风格
    """


def docstrings_func_epytext(parm_a, parm_b, parm_c):
    """
    Epytext 风格

    @param parm_a: 参数a
    @param parm_b: 参数b
    @param parm_c: 参数c
    @return: 结果a
    """


def docstrings_func_restructuredtext(parm_a, parm_b, parm_c):
    """
    reStructuredText 风格

    :param parm_a: 参数a
    :param parm_b: 参数b
    :param parm_c: 参数c
    :return: 结果a
    """


def docstrings_func_numpy(parm_a, parm_b, parm_c):
    """
    NumPy 风格

    Parameters
    ----------
    parm_a : 参数a
    parm_b : 参数b
    parm_c : 参数a

    Returns
    -------
    result_a : 结果a
    """


def docstrings_func_google(parm_a, parm_b, parm_c):
    """
    Google 风格

    Args:
        parm_a: 参数a
        parm_b: 参数b
        parm_c: 参数c

    Returns:
        result_a  结果a
    """
```

### Docstring format 添加参数类型注释

&emsp;&emsp;Python 是动态语言，使用动态类型（Dynamic Typed），即在运行时确定数据类型，变量使用之前不需要类型声明；对于一些已经确定类型的参数，加上类型的注释，可以借助 PyCharm 的方法类型检查功能，在书写代码时就能够提前发现错误。

&emsp;&emsp;下面代码第一行是没加参数类型注释，第二行添加了参数类型注释，PyCharm 就可根据方法对应的 docstrings 提前判断输入参数类型的问题，并给出正确类型提示。

&emsp;&emsp;PyCharm 中开启插入类型占位符注释路径如下： 开启后再使用 Docstring format 添加方法注释，就会出现类型占位符。

> File → to→ Settings → to→ Editor → to→ General → to→ Smart Keys → to→ Insert type placeholders in the documentation comment stub

![Docstring format 添加参数类型注释](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-02.png)

&emsp;&emsp;添加了参数类型的各方法注释如下：

```python
def docstrings_func_epytext_type(parm_a, parm_b, parm_c):
    """
    Epytext 风格 - 参数类型

    @param parm_a: 参数a
    @type parm_a: int
    @param parm_b: 参数b
    @type parm_b: str
    @param parm_c: 参数c
    @type parm_c: bool
    @return: result_a 结果a
    @rtype: int
    """


def docstrings_func_restructuredtext_type(parm_a, parm_b, parm_c):
    """
    reStructuredText 风格 - 参数类型
    
    :param parm_a: 参数a
    :type parm_a: int
    :param parm_b: 参数b
    :type parm_b: str 
    :param parm_c: 参数c 
    :type parm_c: bool 
    :return: result_a 结果a
    :rtype: int
    """


def docstrings_func_restructuredtext_type_2(parm_a, parm_b, parm_c):
    """
    reStructuredText 风格 - 参数类型 与参数描述同一行

    :param int parm_a: 参数a
    :param str parm_b: 参数b
    :param bool parm_c: 参数c
    :return: result_a 结果a
    :rtype: int
    """


def docstrings_func_numpy_type(parm_a, parm_b, parm_c):
    """
    NumPy 风格 - 参数类型

    Parameters
    ----------
    parm_a : int
        参数a
    parm_b : str
        参数b
    parm_c : bool
        参数c

    Returns
    -------
    result_a : int
        结果a
    """


def docstrings_func_google_type(parm_a, parm_b, parm_c):
    """
    Google 风格 - 参数类型

    Args:
        parm_a (int): 参数a
        parm_b (str): 参数b
        parm_c (bool): 参数c

    Returns:
        result_a (int):  结果a
    """

```

### Live Templates

&emsp;&emsp;Docstring format 已经可以自动格式化输出 docstrings，但无法加上创建人、创建时间、修改人、修改时间、版权声明；有些规范建义这些元素写在文件头部，而对于协同开发同一文件，觉得还是需要把这些元素加在各个方法里面，会更清晰明了。

&emsp;&emsp;可通过 PyCharm 的 Live Templates 自定义模板实现。

&emsp;&emsp;Live Templates 中设置路径如下：

&emsp;&emsp;添加方式如下：

1、进入 Live Templates 设置页面，点击右方加号，添加 Template Group

![进入 Live Templates 设置页面](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-03.png)

2、任意名

![新组名](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-04.png)

3、再添加 Live Template

![再添加 Live Template](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-05.png)

4、在下方添加 Abbreviation（快捷键缩写），Desctiption （快捷键描述）

![在下方添加 Abbreviation（快捷键缩写），Desctiption （快捷键描述）](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-06.png)

5、在下方添加 Applicable（可应用的语言范围）

![添加 Applicable](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-07.png)

6、在 Template text 中添加下方模板代码

```python
"""


Parameters
----------

Returns
-------

:Author:  mmy83
:Create:  $DATE$ $TIME$
:Blog:    https:/mmy83.online
mmy83 Group All Rights Reserved.
"""

```

7、在 Edit variables 中配置 DATE 和 TIME，使创建时间自动生成

![Edit variables 中配置 DATE 和 TIME](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-08.png)

8、为了使用注释方便，还可添加 更新时间 和 当前时间

```ini
:update: $DATE$ $TIME$

$DATE$ $TIME$

```

![alt text](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-09.png)

在方法下方输入配置的 Abbreviation，使用 Tab 或 回车都可自动生成注释，见下方代码

![配置 Abbreviation](/images/2024/09/2024-09-24/python的文档注释（Docstring）/python的文档注释（Docstring）-10.png)

### 补充

Live Templates 中可用的宏定义变量有：

```python
# ${PROJECT_NAME} - 项目名称
# ${NAME} - 新建文件的名称
# ${USER} - 当前用户的登录名
# ${DATE} - 当前系统日期
# ${TIME} - 当前系统时间
# ${YEAR} - 当前年份
# ${MONTH} - 当月
# ${DAY} - 当月的当天
# ${HOUR} - 当前小时
# ${MINUTE} - 当前分钟
# ${MONTH_NAME_SHORT} - 月份名称
# ${MONTH_NAME_FULL} - 月份名称
```

> &emsp;&emsp;一般只使用 # -*- coding: utf-8 -*- 作为文件头注释。一是因为某些规范的要求，特别是 PEP8 - Source File Encoding；二是因为非 utf-8 编码情况下，代码可能不识别汉字。
{: .prompt-tip }

## 注意

> 注释不是越多越好。对于一目了然的代码，不需要添加注释。
>
> 对于复杂的操作，应该在操作开始前写上相应的注释。
>
> 对于不是一目了然的代码，应该在代码之后添加注释。
>
> 绝对不要描述代码。一般阅读代码的人都了解Python的语法，只是不知道代码要干什么
{: .prompt-tip }

## 结束语

&emsp;&emsp;程序员总是希望别人写的代码有详细且易于阅读的注释，但是自己写代码的时候却又不爱写注释。嗯，我就是这样的一个程序员。有了这样一个规范可以更好地规范自己的代码，也便于他人阅读。

## 参考资料

1、[文档注释（Docstring）](https://www.kancloud.cn/misstime/python3_notes/678239)

2、[python 注释规范(Docstring) 与 LiveTemplates 介绍](https://blog.csdn.net/qq_46450354/article/details/129737430)

## 感谢

不多说了，说再多也改变不了抄的事实。感谢无私奉献。
