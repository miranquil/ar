---
title: Python+Selenium 入门之在线测眼力
urlname: selenium-eyenurse
date: 2019-05-03 11:54:43
tags:
  - Python
  - Selenium
categories:
  - Algorithm & Programming
---

[游戏链接](http://m.xiaoxineye.com/wxaward/html5/games/eye/index.html)

~~16 岁以下不要再玩手机了！~~

<!--more-->

偶尔看到这个游戏，玩到二十多关发现实在有点变态，但又不甘心，遂产生了破解的念头。一开始是想直接在屏幕上标点标出颜色不一样的格子，但后来想起屏幕画点需要调用系统 API，Mac 下开发的 win 不能用…… 只好换思路。

之后想的是用 PIL 截屏然后分析图像文件，但又总觉得这样做速度会太慢而且杀鸡焉用牛刀。F12 看了下这页面也就是拿 JS 生成了一大堆 HTML 色块，直接读源码就可以分析，那这样的话用类似爬虫的思路不就行了？

单纯的 web 爬虫肯定行不通，因为要模拟点击进入下一关，而每关的色块和答案位置都是不一样的，因此最好的解决方案还是模拟浏览器，遂直接用 Selenium+Safari。

整体还是比较简单，都是一些 Selenium 的基本操作，是个适合入门练手的站。

代码如下，Gist[在这里](https://gist.github.com/yorushika/90490dc0f42a6fef705d9208e0dd6a73)。相对于 Gist 里的内容加了一些说明注释。

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

browser = webdriver.Safari()
browser.set_window_size(1024, 768) # 测试了下大概 1024x768 是最低的能让页面显示完整的 “常见” 尺寸了，大屏党无视。
browser.get('http://m.xiaoxineye.com/wxaward/html5/games/eye/index.html')

# Game start
start_btn = browser.find_element_by_css_selector('button')
start_btn.send_keys(Keys.RETURN)    # 这里其实也可以用 click，如下。

while True:
    color_box = browser.find_element_by_xpath('//div[@id="box"]')
    if not color_box:
        print("No more boxes.")
        break
    box_list = color_box.find_elements_by_xpath('./span')
    color_num_dict = {}
    target_box_dict = {}

    # 查找不同色块，算法是简单的记录出现次数，在发现不同色块出现后只要已经统计的色块数大于 2 即可确定。这里数据结构略繁琐，我觉得应该有更优的做法，望补充。
    for index, box in enumerate(box_list):
        color = box.get_attribute('style')
        if color_num_dict.get(color):
            color_num_dict[color] += 1
            if len(color_num_dict.keys()) == 2 and color_num_dict[color] >= 2:
                break
        else:
            color_num_dict[color] = 1
        target_box_dict[color] = box

    target_box = None

    for key in color_num_dict.keys():
        if color_num_dict[key] == 1:
            target_box = target_box_dict[key]

    if not target_box:
        break

    target_box.click()  # 简单的 click() 动作就好，有趣的是在这里用 keys.RETURN() 是行不通的。初步推断发送回车只适合能用 tab 键定位（虚线括住）的按钮，而对于加了 JS 点击事件的非按钮元素行不通。
```

大概测试了几次，最好成绩是 407 分🦅。

{% asset_img 1.jpg %}

这个游戏后期的颜色差异个人认为已经超过了肉眼感知极限。第 22 关开始不同色块的 RGB 值每项仅仅差了 10，如 `(108, 198, 199), (118, 208, 209)`;42 关开始成了 8，52 关开始到 400 + 关更是降到了 5. 绝大部分人都是卡在了一般的屏幕就已经显示不出差别的 22 关，而 52 关开始几乎不可能认出来，况且 60s 的时间限制也让手慢的人不可能到更高关卡。
