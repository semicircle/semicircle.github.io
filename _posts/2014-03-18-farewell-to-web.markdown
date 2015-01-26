---
layout: post
title: "跟Web说byebye了.."
date: 2014-03-18 18:43
comments: true
categories: web html5 roa soa  
---

这篇文章说的是技术选型问题.

### Sencha Touch

随着深入开发, 感到:
Sencha Touch 实在不适合来开发要长期维护的一个产品, 仅仅适于mock up, 产品原型, 一次性应用.

主要是因为:

1. Sencha Touch 号称流畅程度媲美Native App, 其实是媲美设计不怎么优美的App, 在视觉系App大行其道的今天, 人们的审美标准随之提高, 仅仅用 简陋 + 一般流畅度, 是很难形成一个合格移动互联网产品的基本体验的.

2. 使用 HTML5 开发, 就定义好了多平台兼容的基调, 但实际上, 各个平台的推荐UI设计模式都不尽相同, 生搬硬套一种界面, 会丧失很多用户体验

### RESTful

1. 实在没法RESTful啊, 世界不是那么简单啊, ROA从根本上就没法描述各种服务啊, 现在市面上用RESTful的那些API, 都是直接把 "动词+名词" 一起放在 url 里啊...

2. SOA 有啥错啊? 从SUN RPC 到 SOAP, 不就是SOAP太慢太挫了么, 可Protobuf很快啊...

3. 唯一想得到的, RESTful可以利用Web的中间件, CDN...嗯...可大部分业务逻辑, 也不好用CDN来缓存吧, 要容忍的延迟也太大了吧...

### 总结

最后. RESTful / HTML5 , 一切皆web化, 如果技术方向真是这样, 那通用搜索引擎多高兴啊...  (请自行从阴谋论角度展开) 

ps. 当然, 图片存储, 这些还是必须借助Web的成果的, 各种存储云服务.
