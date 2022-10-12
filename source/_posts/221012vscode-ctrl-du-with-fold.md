---
title: 关于 VSCode 的 Vim 插件在代码折叠时 <C-d> 和 <C-u> 工作不正常
date: 2022-10-12 11:30:23
tags:
  - VSCode
categories:
    - Experience
---

来源：[<C-d\> , <C-u\> not working when code folded #5303](https://github.com/VSCodeVim/Vim/issues/5303)

<!--more-->

参考多个 Issues 发现这东西本质上源于 VSCode 内部 API 对于折叠后代码的翻页操作支持不够。

[prophetw](https://github.com/prophetw) 提供了一个思路：替换原本的 `<C-d>` 和 `<C-u>` 功能为连续指令输入：

```json
{
  "vim.normalModeKeyBindingsNonRecursive": [
    {
      "before": [
      "<C-d>"
      ],
      "after": [
        "1",
        "5",
        "g",
        "j",
        "z",
        "z",
      ]
    },
    {
      "before": [
        "<C-u>"
      ],
      "after": [
        "1",
        "5",
        "g",
        "k",
        "z",
        "z"
      ]
    },
  ],
}

```

本质上是把原本的翻页指令变成了让光标上下跳 15 行之后再 zz 居中。

虽然对于不同分辨率和字号需要重新指定合适的行数，但不失为一良策。

当然，最后我还是没有主力用 VSCode……
