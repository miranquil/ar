---
title: Neovim 退出后终端光标样式变 block 问题
date: 2021-02-28 18:30:05
tags: 
    - Neovim
categories:
    - Experience
---

似乎 Neovim 作者也没有什么特别好的解决方案，在 Neovim 配置文件中固定光标样式可解决（既 Vim 默认的风格）：

```shell
echo "set guicursor=" >> ~/.config/nvim/init.vim
```

如果你的配置文件不是上述路径可能需要调整。

作者还给出了另一解决方案，但较多无效的报告：

```shell
let $NVIM_TUI_ENABLE_CURSOR_SHAPE = 0
```

参考 GitHub Issue：[Nvim changes default cursor style in terminal #6005](https://github.com/neovim/neovim/issues/6005)
