---
title: 开启代理时从 GitHub pull 异常办法
date: 2024-06-18 19:29:04
tags:
  - Github
  - Git
  - Proxy
---

## TL;DR

`vim ~/.ssh/config`

加入以下内容

```shell
Host github.com
  Hostname ssh.github.com
  Port 443
  User git
```

[Reference](https://github.com/vernesong/OpenClash/issues/1960#issuecomment-1115732292)
