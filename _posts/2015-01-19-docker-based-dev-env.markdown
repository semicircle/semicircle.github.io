---
layout: post
title: "(提纲)基于 Docker 的(轮子组装)开发环境"
date: 2015-01-19 07:53:25 +0000
comments: true
categories: docker
---

大半年的时间没有更新了, 主要是因为之前不小心破坏了 gem / bundle 的环境, octopress 全线罢工.

最近利用 Docker 现成的 Image, 玩了很多轮子, 比如  Protobuf, Thrift, Postgres , Openstreemap(我自己安装两三天未果, docker image 下载下来即插即用),

遂, 深深的感到 Docker 的方便之处.

于是将所有的开发环境都放到了 Docker 之中, 

这些使用 Docker 搭建开发环境的过程值得仔细总结一下. 

同时, VagrantUp 这个大坑货也值得好好批判一下. 

- 无限脚本和很多看起来区别不大的轮子

比如, 连小版本都会出错的 rails, 更突显了在二进制备份/统一工作环境的优势.

#(待续).

