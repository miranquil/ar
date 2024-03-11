---
title: 解决 Intellij 系 IDE 在 macOS 下滚动屏幕卡顿
date: 2021-03-07 10:54:10
tags:
    - Intellij
    - macOS
categories:
    - Experience
---

## TL;Dr

**Subpixel** -> **Greyscale**

2024-03-11 UPDATE: 乐，发现现在 macOS 下只剩下 `Greyscale` 一个选项了。

<!--more-->

换双拼后的第一篇文章，老实说还是不熟练。肌肉记忆的养成不是一天半天的事，不过似乎实际速度并没有慢到哪去。而且很神奇的现象是在手机上打得比在电脑上快不少，不知为何在手机键盘上脑子反应键位的速度要快那么些，很神奇，明明用的都是全键盘。

## 前言

一直感觉最近 Clion 的体验不如 Sublime Text 舒服，可又说不出是哪。直到有一次闲得慌乱按 C-d C-u 才发现这滚动时候明显感觉有点卡顿，但又不是特别严重。奈何这种问题不发现则已，一发现就会被带入强迫症想要解决，遂有此文。

~~什么没有慢到哪去……打字明显脑子跟不上手速~~

## 探寻

DuckDuckGo 一发，发现报告这个问题的人还不少。JetBrains 甚至还专门开了个 YouTrack，需要的可自行围观：[JBR-526](https://youtrack.jetbrains.com/issue/JBR-526)。翻了翻其中有说跟分辨率有关，有说和 *code lens* 的配置项有关（这东西有一说一我到现在没明白干嘛的），奈何测试了一下都不太有用。一直翻到有个老哥说更改抗锯齿可以解决，遂尝试。

JetBrains 系 IDE 自带两个抗锯齿模式。**Subpixel** 和 **Greyscale**。这其中还分 IDE 的 UI 用的以及编辑器用的，而 IDE 默认为 **Greyscale**， 编辑器默认为 **Subpixel**。这二者区别我没有细究，单从视觉角度上看 **Subpixel** 相对来说字体更为圆润，视觉效果比 **Greyscale** 略粗一些。另外我手上没有别的系统无法测试，目前 macOS 下 IDE 只支持 Greyscale，但我好像有其他系统下同时支持两者的印象。

~~这里顺便提一嘴，JetBrains 是唯一一个能在某爱国云桌面提供的辣眼睛画面下做到清晰平滑的。~~

然而有个很特殊的情况是我目前用的 LG 4K 屏幕（HDR 关闭）的分辨率被我开了缩放，也就是把 4K 分辨率缩放至 2K 来提高清晰度（macOS 其实更狠，默认分辨率是缩到 1080P）。而这个情况下，这两种抗锯齿模式根本……看不出多大区别。

于是乎，把 Editor 的抗锯齿也改成 **Greyscale** 圆满解决，实际的显示效果，字母感觉微微瘦了那么——一丝，稍微拉远点距离就看不出什么区别（实际上我已经离得很近了都只能勉强看出切换时的变化）。不过如果是用的显示器原生分辨率而非缩放的话，我测试了一下这两个区别还是相当明显，**Subpixel** 明显更顺眼。但悲哀的是，即使是原生分辨率下 **Subpixel** 还是会卡顿……

## Reference

关于 **Subpixel** 次像素抗锯齿和 **Greyscale** 灰度抗锯齿，我去 6o 的 TG 群里问了一下，有大佬给了两篇科普文章：

* [macOS Mojave 字体渲染由默认的灰度抗锯齿改回之前的次像素抗锯齿](https://lvii.github.io/system/2018-09-26-setting-macos-mojave-font-rendering-from-grayscale-to-subpixel-antialiasing/)
* [视觉分辨力与 Retina Display](https://www.thetype.com/2010/06/2619/)
