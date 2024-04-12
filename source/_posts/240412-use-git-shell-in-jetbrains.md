---
title: 在 JetBrains 系列 IDE 中使用 Git Bash/Sh
date: 2024-04-12 15:48:51
tags:
  - JetBrains
  - Shell
categories:
  - Algorithm & Programming
---

## TL;DR

1. File -> Settings -> Tools -> Terminal
2. `Shell path` -> `(Path to git)\bin\bash.exe --login`

<!--more-->

被 VDI 里 Windows 的 Powershell 搞得心力憔悴。不支持诸多 Ctrl 组合键，配合 VDI 本身就有点不稳定的延迟卡顿更是难以忍受。

别问我为什么不直接用 Windows Terminal，那东西甚至天生自带延迟。

某天突然想起来 Windows 下的 Git 自带有 Git Bash，添加进上面的配置发现一启动终端就会跳出个黑框，才想起来这玩意是个 win33 的 GUI……不甘心之余，想起这东西运行时候也是套了个 win 命令行程序的外框，那没准实际上本身也只是套壳而已。于是翻了一下 Git 的安装目录。

果不尽然，在安装目录下发现一个 bin 文件夹，里面摆着 git 本尊和另外两个眼熟的东西……`sh` 和 `bash`，戳开发现就是想要的。

于是加进 Goland，别忘了打上 `--login`参数，效果完美，又可以愉快的用习惯的 Unix-like 快捷键了。
