---
layout: post
title: "Sencha Touch: 在Contorller间通信的最佳实现方法"
date: 2013-10-10 17:24
comments: true
categories: sencha html5
---

这个技巧在以前的blog里用英文写了一遍, 后来发现实在有用, 又容易忘, 就再用中文转述一遍, 增强记忆. 正文:

这个技巧是从文档中拼凑出来的, 没有专门说明的文章.

### 事件比直接调用更好

一种简单的方法: 先获取Controller对象`this.getApplication().getController()`, 再直接调方法. 但这种方法会让两个Controller耦合过于严重.

### 全局事件

在Sencha的文档中, 没有发现事件总线之类的设计, 所有事件都是分散的, 如何实现全局事件呢?

如下:

```js
this.getApplication().on('MyGlobalEvent', function(){});

this.getApplication().fireEvent('MyGlobalEvent', ...);
```
