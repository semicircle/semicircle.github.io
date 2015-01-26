---
layout: post
title: "(提纲)Apache Thrift 爬坑行"
date: 2015-01-19 08:16:53 +0000
comments: true
categories: thrift
---

Apache Thrift 爬坑行 (兼与 Protocol Buffer 的比较) (提纲)

### 版本
1. thrift 0.9.2

### 编译相关的常见误解

1. make thrift complier 与 lib 的区别, lib 其实不用编译, 很多现成途径获取.
2. mac 环境: 一年多没解决了, 省省吧.
3. mac 环境不支持但能从 Linux 编译 .thrift 生成 Cocoa 代码? ---就是这么跳脱

### 前向声明在哪里

别找了, 没有的. 怎么办?

### 奇怪的底层协议

每次请求并非一次通信

### 某些 Client 不支持异步

经典 RPC 纯爷们儿, 就 TM 不向 RESTful 低头! 

- 那你倒是支持异步啊, 
- 有人说Callback-Hell众口难调, 

- 那咱支持个 Future, 没 Callback 了吧?
- 逼格太高

- 那咱支持个设超时?
- ...
