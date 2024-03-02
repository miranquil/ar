---
title: 制作一个 pprof 的 Docker 镜像
date: 2024-03-01 17:01:29
tags: 
  - Golang
  - Docker
categories:
  - Algorithm & Programming
---

最近在研究 pprof 性能测试，考虑到公司很多项目开发和测试环境都部署在内网，而 pprof 有诸多依赖 ~~graphviz 别看了说的就是你~~，于是便打算用 Docker 封装一份 pprof 做成开箱即用。

<!--more-->

## 几个坑

* 导入镜像的 Go 与 Docker 宿主机架构保持一致~~老生常谈~~
* graphviz 在 `Centos7` 中默认安装，但版本不够新，需要 `yum update` 后再次安装。
  * 否则实际运行时会报 `Failed to execute dot. Is Graphviz installed? exec: "dot": executable file not found in $PATH`，有点搞笑。
  * 好在 `CentOS` 下这东西安装比 `homebrew` 快多了，不过后者安装时没有和以前一样报依赖问题也是庆幸。
* 如果额外安装独立的 `pprof@latest` 注意路径。这东西会安装在当前用户目录的 `go/bin/` 下。

## Dockerfile

```dockerfile
FROM centos:7
LABEL version="0.1" author="shijianning"

RUN yum update
RUN yum install -y graphviz

WORKDIR /root
COPY go1.21.7.linux-amd64.tar.gz ./
RUN tar -C /usr/local -xzf go1.21.7.linux-amd64.tar.gz
RUN rm go1.21.7.linux-amd64.tar.gz
ENV PATH=$PATH:/usr/local/go/bin
ENV GOPROXY=https://mirrors.aliyun.com/goproxy/
RUN ["go", "env", "-w", "GO111MODULE=on"]
RUN ["go", "env", "-w", "GOPROXY=https://goproxy.cn,direct"]
RUN ["go", "install", "github.com/google/pprof@latest"]
RUN ["mv", "/root/go/bin/pprof", "/usr/local/go/bin/"]
RUN ["rm", "-rf", "/root/go"]
ENTRYPOINT ["pprof"]
```
