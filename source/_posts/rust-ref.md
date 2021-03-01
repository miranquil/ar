---
title: Rust By Example 笔记：ref 关键字与 & 的区别
date: 2021-03-01 21:47:56
tags:
    - Rust
    - Rust By Example
categories:
    - Algorithm & Programming
---

参考文档：[ref - Rust](https://doc.rust-lang.org/std/keyword.ref.html)

`ref` 用于在模式匹配中**借用**值而非夺走它的所有权，由于在 `match` 表达式中变量会被 *consumed*。**对于没有实现 `Copy trait` 的类型，若不使用 `ref` 会导致后续无法使用之前的变量。**

> By default, identifier patterns bind a variable to a copy of or move from the matched value depending on whether the matched value implements `Copy`. 
> https://doc.rust-lang.org/reference/patterns.html#identifier-patterns

```rust
let maybe_name = Some(String::from("Alice"));
// The variable 'maybe_name' is consumed here ...
match maybe_name {
    Some(n) => println!("Hello, {}", n),
    _ => println!("Hello, world"),
}
// ... and is now unavailable.
println!("Hello again, {}", maybe_name.unwrap_or("world".into()));
// error[E0382]: use of partially moved value: `maybe_name`
```

使用 `ref` 关键字就可仅借用变量 n 并维持其所有权。

```rust
let maybe_name = Some(String::from("Alice"));
// Using `ref`, the value is borrowed, not moved ...
match maybe_name {
    Some(ref n) => println!("Hello, {}", n),
    _ => println!("Hello, world"),
}
// ... so it's available here!
println!("Hello again, {}", maybe_name.unwrap_or("world".into()));
```

注意这里虽然 `ref` 是标在了 `n` 前面，但其对应的是 `maybe_name`。

`ref` 关键字出现的位置与 `&` 非常类似，但二者用途不同。上述代码中将对应位置替换为 `&` 会报错。


