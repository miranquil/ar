---
title: 使用pickle来在Python中序列化数据
date: 2019-05-22 17:05:48
urlname: pickle-serialize-in-python
tags:
  - Python
categories:
  - Algorithm & Programming
---

参考[官方文档](https://docs.python.org/3/library/pickle.html#pickle-protocols)。
pickle 模块提供了将Python对象序列化的功能。相对于流行的 JSON，pickle无法在 Python 以外的环境中使用，但对于对象的适应力更强。适用于数据抽象、加密、存储等场景。

**官方警告**：pickle 模块在接受被错误地构造或者被恶意地构造的数据时不安全。永远不要 unpickle 来自于不受信任的或者未经验证的来源的数据。

<!--more-->

在 Python 中其实一共有两个数据序列化的包，一个是 pickle，另一个则是 [marshal](https://docs.python.org/3/library/marshal.html#module-marshal)。不过后者主要是用来支持 Python 的 `.pyc` 缓存而非开发使用，这里不多介绍，关于 pickle 和 marshal 的对比也略过，有兴趣的可以参考上面的链接。

## pickle vs JSON

* JSON 是文本序列格式，绝大部分情况都是 `UTF-8` 编码。而 pickle 则是二进制格式。
* JSON 对人类阅读友好，pickle 嘛，嗯……
* JSON 可在广泛的领域中自由使用，而 pickle 则仅限于 Python 内部。
* 重头戏：JSON 通常情况下只能表达有限的 Python 内建类型，碰见用户自定义类就束手无策。而 pickle 可以表达巨量的 Python 类型——绝大部分还是自动的，如果你真的搞了个大新闻，还可以接入 [特定对象 API](https://docs.python.org/3/library/pickle.html#pickle-inst)。

## 数据流格式

由于 pickle 应用于 Python 内部，它不具有 JSON 或 XDR 等的诸多限制，虽然那也意味着非 Python 程序无法利用，不过这个东西还是看需求。

通常来说 pickle 会对数据进行相对的压缩，如果你还有额外需求，对生成的数据进行[进一步压缩](https://docs.python.org/zh-cn/3/library/archiving.html)也是很容易的，毕竟说破大天它也就是个 `bytes` 流。

此外 [pickletools](https://docs.python.org/3/library/pickletools.html#module-pickletools) 对 pickle 生成的数据流提供了更多的分析工具，这里略过。

由于一些历史原因~~你懂得~~，pickle 的序列化过程有五种协议可用。协议版本越高，对 Python 的版本要求就越高。

* 版本 0 是最早的“人类可读”协议，和更早版本的 Python 向后兼容。
* 版本 1 是同样兼容早期版本 Python 的旧二进制格式。
* 版本 2 在 Python 2.3 推出。它对新型类的速度更快。你可以参考 [PEP 307](https://www.python.org/dev/peps/pep-0307) 来获得更多内容。
* 版本 3 在 Python 3.0 被添加。支持 `bytes` 对象，并且无法被 Python 2.x 版本解码。这是现在的默认协议，建议在要求和其他 Python 3 版本兼容时使用。
* 版本 4 在 Python 3.4 推出，添加了对大型对象的支持并且可以序列化更多类型，对于数据格式也进行了一些优化。你可以参考 [PEP 3154](https://www.python.org/dev/peps/pep-3154)来看看更多的版本 4 协议的增强内容。

## 模块接口

简单的使用 `dumps()` 函数就可以将对象序列化，而 `loads()` 则相反。如果你有更多需求，也可以分别创建 `Pickler` 或者 `Unpickler` 对象。

`pickle` 模块提供下列常量：

* `pickle.HIGHEST_PROTOCOL`
    一个整数，为目前可用的最高协议版本。这个值可以被作为 *protocol* 参数传递给 `dump()` 和 `dumps()`，以及 `Pickler` 的构造器。
* `pickle.DEFAULT_PROTOCOL`
    一个整数，执行操作时使用的缺省协议版本，可以小于 `HIGHEST_PROTOCOL`。当前默认值是3，是一个为 Python 3 设计的新协议。

下面是执行操作用到的函数：

### pickle.dump(obj, file, protocol=None, *, fix_imports=True)

将序列化的对象写入打开的文件。这个函数与 `Pickler(file, protocol).dump(obj)` 同理。
*protocol* 参数指定使用的协议版本，支持从 0 到 `HIGHEST_PROTOCOL`。如果没有指定，缺省为 `DEFAULT_PROTOCOL`。如果是负数就会直接视为 `HIGHEST_PROTOCOL`。
*file* 参数必须拥有能接收单 bytes 序列的 `write()` 方法。它可以是磁盘上以二进制写打开的文件、一个 `io.BytesIO` 实例，或者任何符合接口的自定义对象。
*fix_imports* 为 `True` 并且 *protocol* 小于 3 时，pickle 会尝试映射新的 Python 3 类到 Python 2 里旧的类名来让数据流对于 Python 2 也可用。

### pickle.dumps(obj, protocol=None, *, fix_imports=True)

以 [bytes](https://docs.python.org/3/library/stdtypes.html#bytes) 返回序列化的对象，而不是写入文件。
参数 *protocol* 和 *fix_imports* 与 `dump()` 中的用法相同。

### pickle.load(file, *, fix_imports=True, encoding="ASCII", errors="strict")

从文件中读取已经被 pickle 序列化的对象并返回。这个函数与 `Unpickler(file).load()` 同理。
被读取内容所使用的 pickle 协议版本会自动探测，所以不需要加 *protocol* 参数。超出 pickle 对象的 bytes会被自动忽略。
*file* 参数必须拥有两个方法：接收整型参数的 `method()` 方法和无需参数的 `readline()` 方法。这两个方法都应该返回 bytes 类型。因此 *file* 参数可以是磁盘上以二进制读打开的文件、一个 `io.BytesIO` 对象，或者任何符合接口的自定义对象。
可选参数有 *fix_imports*、*encoding* 和 *errors*，它们在兼容由 Python 2 生成的 pickle 流时很有用。*fix_imports* 为 `True` 时 pickle 会映射老 Python 2 类为 Python 3 里的新类名。*encoding* 和 *errors* 告诉 pickle 怎么解码 Python 2 生成的 8 位字符串；它们的默认值分别是 `ASCII` 和 `strict`。*encoding* 参数也可以设置为 `bytes` 来将这些8位字符串读取为 bytes 对象。当解码 Python 2 生成的 *NumPy* 数组和 *datetime*、*date* 和 *time* 序列化实例时，*encoding* 必须为`latin1`。

pickle 模块定义了三个异常：

### exception pickle.PickleError

其他异常类的基类，继承了 `Exception`。

### exception pickle.PicklingError

当对象无法使用 pickler 序列化时抛出的异常，继承了 `PickleError`。

参考[什么可以被序列化](/#什么可以被序列化)来确定适合的对象。

### exception pickle.UnpicklingError

解码出错如数据篡改或不安全操作时抛出的异常。继承了 `PickleError`。

需要注意的是在解码时也可能抛出其他异常，包括（但不仅限于）`AttributeError`、`EOFError`、`ImportError` 和 `IndexError`。

上述内容已经足够应对常见的序列化任务，pickle 模块还包含两个类，`Pickler` 和 `Unpickler`。具体不再介绍，有兴趣的童鞋可以进一步参考[官方文档](https://docs.python.org/3/library/pickle.html#pickle.Pickler)。