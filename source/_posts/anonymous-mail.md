---
title: 关于匿名邮箱的那点事
date: 2021-01-24 17:34:57
tags:
  - Anonymous
categories:
  - Expereience
---

这篇文章其实早应该有了，奈何当时看见相关内容的时候我忘记了在做什么总之是没有第一时间响应，事后那篇文章也找不到了，无奈自己又做了多余功课。

## TL;DR

[Temp Mail - 临时性 - 匿名电子邮件](https://temp-mail.org/zh/)

[10 Minute Mail](https://temp-mail.org/zh/10minutemail)

[altmails | Disposable anonymous email alias forwarding service](https://altmails.com)

<!--more-->

之前看到有人在 v2ex 上讨论 macOS 版 QQ 最近版本总是跳出个截图功能失效的对话框，这玩意儿只要关掉截图热键就会出现，而且不能 Esc 掉就很烦。一票帖子吐槽无数但却没啥办法也是无奈。后来楼主老哥用 time machine 还原了旧版本的安装包，我回帖请他提供一个被问到邮箱，于是这才想起来。公共场合哪怕是 v2ex 这样的论坛让我留邮箱心里还真是没底，于是就有了上面提到的补课……

翻遍手机里的 Telegram、知乎、微信也没找到之前到底是在哪看的内容，印象里是介绍了一家免费转发邮件的服务：申请一个免费的邮箱账号，可以把这个账号绑定到你现有的邮箱里。一旦这个帐号收到邮件就会自动转发给绑定邮箱，同时还可以随时销毁这个帐号。搜了一圈知乎，找到的要么收费（mailbox）要么就是域名都已经卖了要么干脆打不开（亏我还挂着个香港的 FINAL）……

Google 了一下发现一家做代理服务的博文，众所周知这种服务商一般都会多少介绍这种服务（某安全上网频道笑而不语），这里面介绍了一家测试了一下还不错：[Temp Mail - 临时性 - 匿名电子邮件](https://temp-mail.org/zh/)。支持中文，国内可直接打开也是一大亮点。

点开后一目了然，页面正上方是系统分配的邮箱账号，下方是熟悉的收件箱。这个帐号我测试了一下是根据 Cookies 来分配的，一旦进入网站就会分配一个 Cookies，今后无论怎么刷新也都是给你提供这一个账号的内容。另外也可以手动变更地址。猜测如果输入一个固定的地址就可以实现固定账号服务？具体我没有进一步测试。

另外他们家还有提供 10 分钟临时邮箱服务 [10 Minute Mail](https://temp-mail.org/zh/10minutemail)，功能与普通一样但账号存在时间只有 10 分钟。此外 Temp Mail 还提供收费的 Premium 服务，我就没有细看了。

不过似乎和之前印象里的自动转发不太一样？答案在 [altmails | Disposable anonymous email alias forwarding service](https://altmails.com) 这里。这家似乎和上面那个是合作关系，要么就是同一家，毕竟主页有他们的引导链接。输入想要的 Account Name 与自己的邮箱，点击 Create 后会发送一封带验证链接的验证邮件。我测试了一下，用 outlook 邮箱发给匿名绑定的 Gmail，延迟只有一分钟左右，并且没有被归并到垃圾邮件。

**需要注意的是，在点击 Create 时会有一个 Google 的 reCAPTCHA 验证，墙内用户悲。**

收到的邮件其实是以转发的形式发给绑定邮箱的，可以直接回复。另外每一封转发的邮件都提供了 Unsubscribe 按钮，可以随时解除当前的绑定关系。

不过需要注意的是一个邮箱地址只能同时存在一个绑定关系，如果需要第二个 altmail 地址则需要先 unsubscribe 上一个，这和我印象里看到的那篇介绍不太一样。目前还没有找到更好的替代品，好事多磨。
