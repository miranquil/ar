---
title: 220909 关于 Go 的 defer
date: 2022-09-09 11:34:41
tags:
  - Golang
categories:
    - Algorithm & Programming
---

业务中难免遇到这样的代码：

```go
tx := db.Begin()

commit := false
defer func() {
    if commit {
        tx.Commit()
    } else {
        tx.Rollback()
    }
}
// ...
commit = true
```

此处的 `commit` 虽然同样属于当前函数的作用域，但也违背了“不要使用共享内存的方式通信“。

但如果使用额外的单位通道来存储一个 `bool` 又显得多余。

只能说 defer 注定了某些时候的自相矛盾？
