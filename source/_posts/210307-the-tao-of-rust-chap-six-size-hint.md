---
title: 《Rust 编程之道》第六章关于 size_hint 的疑问
date: 2021-03-07 21:20:45
tags:
---
原书中对此方法的返回结果 `(usize, Option<usize>)` 解释为：

> 此元组表示迭代器剩余长度的边界信息。元组中第一个元素表示下界，第二个元素表示上界……代表已知上界或者上界超过 usize 的最大取值范围……size_hint 方法实际返回的是迭代器起点指针到终点指针的距离值。

的确不太能理解这里的**上界**和**下界**到底是指什么的上下界，最后的**距离值**也没能理解。

<!--more-->

方法官方源码：[Trait std::iter::Iterator](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.size_hint)。

文档给出了一段代码示例

```Rust
// Basic usage:

let a = [1, 2, 3];
let iter = a.iter();

assert_eq!((3, Some(3)), iter.size_hint());


// A more complex example:

// The even numbers from zero to ten.
let iter = (0..10).filter(|x| x % 2 == 0);

// We might iterate from zero to ten times. Knowing that it's five
// exactly wouldn't be possible without executing filter().
assert_eq!((0, Some(10)), iter.size_hint());

// Let's add five more numbers with chain()
let iter = (0..10).filter(|x| x % 2 == 0).chain(15..20);
// now both bounds are increased by five
assert_eq!((5, Some(15)), iter.size_hint());
```

Example 倒是能看明白，对于一个由 `.iter()` 生成的 `iter<i32>` 对象，它的 `size_hint()` 返回值是两个总长度组成的元组，但这最后的 `(5, Some(15))` 代表什么却不得而知。

几轮简单调试下来发现个规律，不同方式生成的迭代器即使内容相同，该方法结果也未必一样。如单纯从`(a..b).into_iter()`转换：

```Rust
assert_eq!((5, Some(5)), (15..20).into_iter().size_hint());
assert_eq!((5, Some(5)), (0..5).into_iter().size_hint());
```

而如果是由现有迭代器调用 *filter* 方法生成：

```Rust
assert_eq!((0, Some(10)), (0..10).filter(|_|true).size_hint())
```

再看 *Filter* 的 `size_hint()` 对应[源码](https://doc.rust-lang.org/src/core/iter/adapters/filter.rs.html#59)：

```Rust
    #[inline]
    fn size_hint(&self) -> (usize, Option<usize>) {
        let (_, upper) = self.iter.size_hint();
        (0, upper) // can't know a lower bound, due to the predicate
    }
```

这里能明白为何 *filter* 产生的迭代器下界一直是 **0**。

至于为何 *chain* 产生的上下界都如此诡异，还是得看[源码](https://doc.rust-lang.org/src/core/iter/adapters/chain.rs.html#192)。~~顺便吹一句，Intellij 好强，Clion 对 Rust 的调试能力不是其他工具能比的。~~

```Rust
    #[inline]
    fn size_hint(&self) -> (usize, Option<usize>) {
        match self {
            Chain { a: Some(a), b: Some(b) } => {
                let (a_lower, a_upper) = a.size_hint();
                let (b_lower, b_upper) = b.size_hint();

                let lower = a_lower.saturating_add(b_lower);

                let upper = match (a_upper, b_upper) {
                    (Some(x), Some(y)) => x.checked_add(y),
                    _ => None,
                };

                (lower, upper)
            }
            Chain { a: Some(a), b: None } => a.size_hint(),
            Chain { a: None, b: Some(b) } => b.size_hint(),
            Chain { a: None, b: None } => (0, Some(0)),
        }
    }
```

所以原因是 *chain* 后的对象将两个对象对应的 `size_hint()` 进行了相加。但这两个值其定义究竟为何目前还是不明。粗略搜索似乎这个方法在具体实现时需要开发者保证可靠度，否则会影响对应 *Iterator trait* 的实现。具体原理还需继续挖掘。
