---
title: Python functools 包的一些姿势
urlname: functools-gestures
date: 2019-05-22 10:54:50
tags:
  - Python
categories:
  - Algorithm & Programming
---
借[官方文档](https://docs.python.org/zh-cn/3.7/library/functools.html)的话说：
> functools 模块应用于高阶函数，即——参数或（和）返回值为其他函数的函数。通常来说，此模块的功能适用于所有可调用对象。

这里记录一些个人需求用到的内容。


<!--more-->
## functools.partial

> Return a new partial object which when called will behave like func called with the positional arguments args and keyword arguments keywords. If more arguments are supplied to the call, they are appended to args. If additional keyword arguments are supplied, they extend and override keywords.

简单来说，partial会返回一个“客制化”的对象，直接call会得到对应func执行的结果，其中传入的参数可以进行客制化。

partial 的原理大概可以粗略的这样表示：

```Python
def partial(func, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = keywords.copy()
        newkeywords.update(fkeywords)
        return func(*args, *fargs, **newkeywords)
    # 下面三行如果不明所以然可以直接无视
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```

第一次看见带星号的有点迷糊？简单来说就是一个函数调用比如 `output(1, 2, c=3, d=4)`，这里 *func* 就是 `output`，*args* 就是 `(1, 2)`，*keywords* 就是字典 `{'c': 3, 'd': 4}`。partial 函数会定义一个新的函数 `newfunc`。在调用 `newfunc` 时传入下列内容：

1. 调用 `newfunc` 时传入的没有指定参数名的参数，按照函数定义的参数表列赋值。
2. 使用 `partial` 时传入的没有指定参数名的参数，即上文中的 *args*，同样按照函数定义的参数表列赋值。
3. 使用 `partial` 时传入的指定变量名的参数，即上文中的 *keywords*，该参数表中如果存在第一步中赋过值的参数，该值会**在该次操作时**被 *keywords* 中的值覆盖。
另外传入参数个数不得超过原函数的参数个数（毕竟最后实质上还是执行了原参数）。

还是太抽象？比如我有下面的定义：

```python
def output(a, b, c, d):
    print('a={} b={} c={} d={}'.format(a, b, c, d))

o1 = partial(output, 1)
o2 = partial(output, d=4)
o3 = partial(output, b=2)

o1(2, 3, 4)
# 输出 a=1 b=2 c=3 d=4
o2(1, 2, 3)
# 输出 a=1 b=2 c=3 d=4
o3(1, c=3, d=4)
# 输出 a=1 b=2 c=3 d=4
o3(2, b=3, c=4, d=5)
# 输出 a=2 b=3 c=4 d=5
o3(1, 3, 4)
# TypeError: output() got multiple values for argument 'b'
o3(1, 2, 3, 4)
# TypeError: output() got multiple values for argument 'b'
```

注意后面 o3 的例子。由于在参数表列位置第二的 b 变量被 partial 指定为 2，因此往后如果**直接**传入参数 c 和参数 d 的值而不指定参数名，则会由于重复对参数 b 赋值导致出错，如同函数定义时一样，默认参数永远靠后。

另外参考上文中的第三条，`o3(2, b=3, c=4, d=5)`这步操作由于参数表指定了参数 b 的新值，因此即使在定义 o3 时指定了参数 b 的值为 2，新值 3 也会在执行时覆盖原有的定义（仅限本次操作，不影响下次调用 o3）。

其实换一种角度更容易理解，以第四行 `o3(2, b=3, c=4, d=5)`举例。

1. 根据 o3 的定义，参数 b 的值为2，其他参数未知。此时参数表为 {a:? b:2 c:? d:?}。
2. 依照原函数 output 的参数表顺序，传入未指定参数名的参数（只有一个 `2`，按顺序为参数 `a`）。此时参数表为 {a:2 b:2 c:? d:?}。
3. 传入其他**已经指定参数名**的参数，即 `(b=3, c=4, d=5)`。此时参数表中 b 的值被新值取代，其他参数也逐个更新。为 {a:2 b:3 c:4 d:5}。
4. 依照最新的参数表 {a:2 b:3 c:4 d:5}，将参数代入原函数 `output` 执行，即 `output(2, 3, 4, 5)`。

以下是官方给的一个实践例子：

```python
>>> from functools import partial
>>> basetwo = partial(int, base=2)
>>> basetwo.__doc__ = 'Convert base 2 string to an int.'
>>> basetwo('10010')
18
```

同样，若我们在调用的时候重新为 base 赋值，如`basetwo('10010', base=10)`，结果自然是10010。
