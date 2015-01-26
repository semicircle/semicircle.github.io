---
layout: post
title: "Sencha Touch 在Chrome中调试的技巧"
date: 2013-04-11 12:16
comments: true
categories: html5 sencha
---

简要介绍几种在实践过程中发现的Chrome Developer Tools技巧

### 在初始化过程加Breakpoint

在development环境下, 为了防止浏览器Cache js文件, 影响"修改/看结果"的调试过程. ExtJs的micro loader 在请求js的url后, 会增加形如?_[timestamp]的参数. 这本身是方便调试的功能, 但如果你想用Developer Tools 把断点加在初始化过程, 问题就来了, 因为timestamp这个参数, Chrome会认为你增加的断点, 与刷新后加载的js不是同一个文件, 解决办法分两部:

####### STEP1. 关掉Sencha Touch的timestamp

在app.js 首部增加:

``` js
Ext.Loader.setConfig({
    disableCaching: false
});
```

###### STEP2. 关掉Chrome的Cache功能

首先打开Developer Tools的Settings (通过右下角的菊花打开, markdown怎么加截图啊...)

Settings -> General -> Diable cache , check it.

然后就一切正常了.

<!-- more -->

### 用Console 刺探 对象

可以利用AppName的对象, 假设你的Sencha App起名为DemoApp

```
	DemoApp.what = this
```

然后, 你就可以在 Console里, 通过```DemoApp.what.```这个名字肆意亵玩这个this了~

这个技巧在某个Sencha视频中也有提到, 实践中是非常有效的技巧. 属于熟悉Sencha的第三大手段~ (前两大是文档和源代码)

### DOM Breakpoint

有时界面上有些元素, 不知在什么时候, 被什么事件和代码所修改.

通过在Element 的 tab里, 在想添加DOM Breakpoint的地方点右键, 可以设置各种各样的触发条件, 很方便.

### 警惕 Chrome Extension.

很多Chrome Extension 会在 DOM里添加元素, 在Sencha Touch管辖的寸土寸金的地盘上, 有些元素可能对界面具有破坏效果.

已知黑名单:
1. StayFocused
2. Xunlei
3. smooth scroll



