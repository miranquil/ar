---
title: Apple 设备线缆及充电相关
date: 2021-01-16 18:02:14
tags:
  - Apple
---

这几天收拾房间整理出来一些数据线，新的旧的都有，甚至有几根拆了包装就没有用过。随后翻了一下订单整理了点参数一类的东西，这里留个记录以供参考。

<!--more-->

不同的Apple设备其对应充电功率也不一样（手机类似安卓的高功率先别想了）。苹果有一篇专门讲这个的文章[《为Mac笔记本电脑找到正确的电源适配器和线缆》](https://support.apple.com/zh-cn/HT201700)。

这句话很关键：

> Mac 笔记本电脑的电源适配器有 29W、30W、45W、60W、61W、85W、87W 和 96W 八种功率规格。您应根据自己的 Mac 笔记本电脑使用相应功率的电源适配器。使用功率更高的兼容电源适配器不会引起任何问题，但这样并不会提高电脑的充电速度，也不会改变电脑的运行情况。如果您所用电源适配器的功率低于 Mac 随附适配器的功率，则适配器将无法为电脑提供足够的功率。

## MacBook Pro

从我接触到的来看，MacBook Pro 13（2018）使用的是61W的电源适配器，而 MacBook Pro 16（2020非M1）却使用了96W的电源适配器。新款向下兼容，但旧款电源适配器无法提供足够的功率给新款使用。

电源适配器的输出功率是选择的第一个指标。只要有合适的线缆，MacBook大块头电源适配器甚至可以拿来充iPhone，同理iPad。当然，功率过高会导致充电时设备温度高于低功率充电，不过也没什么严重影响。

## iPhone

与以往的18W不同，iPhone 12系列将快充需求提高到了20W。

> 若要快速充电，iPhone 12 机型需要最小输出功率为 20 瓦的电源适配器，如 Apple 20W USB 电源适配器。

在有使用合适的线缆的情况下，iPhone 8 及更高机型可激活快速充电（30分钟内充满约50%的电量）。但电源适配器必须提供足够的功率（除12系列外需18W）。Apple 18W、20W2、29W、30W、61W、87W 或 96W USB-C 电源适配器均可用于给iPhone直接快速充电（可看出这几款其中包含了iPad甚至MacBook的电源适配器）。

# iPad

机型众多，一言以蔽之：使用输出功率大于等于随机附赠的电源适配器即可。

> 如果您有瓦数更高的 USB-C 电源适配器（例如 Mac 笔记本电脑随附的电源适配器），您可以将它与 iPad Pro 搭配使用，这时您可能会发现充电速度更快。您也可以将 iPad Pro 连接到电脑上的 USB-C 端口来为它充电。

~~感慨一句，iPad Pro本身也可以拿来给其他设备充电，甚至可以两台iPad Pro互相充，只需要重连就可以切换（捂脸）~~

## 线缆

~~其实主要是想写这个。。。但还是想扩展下其他内容~~

根据对应设备的不同，Apple设备购买时附赠的线不一定支持大功率快充。最简单的识别方式是看**线身上的序列号**。好了，别惊讶，的确很不显眼，但仔细看的确有。

> 线缆的序列号印在它的外壳上，在“Designed by Apple in California. Assembled in China.”字样的旁边。

* 如果序列号的前三个字符是 C4M 或 FL4，则表示线缆可与最高可达 61W 的 Apple USB-C 电源适配器搭配使用。
* 如果序列号的前三个字符是 DLC、CTC、FTL 或 G0J，则表示线缆可与最高可达 100W 的 Apple USB-C 电源适配器搭配使用。
* 如果线缆上有“Designed by Apple in California. Assembled in China”字样，但没有序列号，则您可能符合[更换 USB-C 充电线](https://www.apple.com/support/usbc-chargecable/)的条件。

我手边的设备可供参考的有：

* 11 英寸 iPad Pro（第 4 代）：20W USB-C，`FIL`开头USB-C线。
* iPhone 12 Pro：`FIL`开头 Apple USB-C 转闪电连接线。
* 16英寸 MacBook Pro 2020: `FIL`开头USB-C线。**值得注意的是，这根线比其他配套送的线更粗**
* AirPods Pro：*Assembled in Vietnam* `FGJ`开头 Apple USB-C 转闪电连接线。**最大功率未知**。

要注意的是，如果需要给MacBook这种大功率设备充电，使用的线缆必须能够提供足够的功率（即上文中的DLC、CTC、FTL或G0J序列号）。如果是第三方的线，购买之前务必确认。

## 协议

苹果系列设备对于充电协议非常挑————只认一个PD。适配安卓的快充适配器如果不支持PD协议那在iPhone上的输出功率只能有10W，还不如iPad Pro 2018的18W头。因此购买第三方充电适配器时认准PD关键字，充电宝同理。

## 参考

* [通过 iPad Pro 上的 USB-C 端口进行充电和连接](https://support.apple.com/zh-cn/HT209186#chargeout)
* [适用于 iPhone 的电源适配器](https://support.apple.com/zh-cn/guide/iphone/iph8c1e31583/ios)
* [为 Mac 笔记本电脑找到正确的电源适配器和线缆](https://support.apple.com/zh-cn/HT201700)
* [关于 Apple USB 电源适配器](https://support.apple.com/zh-cn/HT210133#safety)
* [iPad USB 电源适配器认证](https://support.apple.com/zh-cn/HT202916)
* [用 iPad 或 Mac 笔记本电脑电源适配器为 iPhone 充电](https://support.apple.com/zh-cn/HT202105)
* [为 iPhone 快速充电](https://support.apple.com/zh-cn/HT208137)
* [关于 Apple USB-C 至闪电连接线](https://support.apple.com/zh-cn/HT205807)
* [用 iPad 或 Mac 笔记本电脑电源适配器为 iPhone 充电](https://support.apple.com/zh-cn/HT202105)
* [如何优雅地给iOS设备快速充电（iPhone、iPad 快充全攻略）](https://zhuanlan.zhihu.com/p/61774719)
