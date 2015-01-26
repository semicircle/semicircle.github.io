---
layout: post
title: "sencha touch Animation的注意事项"
date: 2013-04-12 18:53
comments: true
categories: html5 sencha
---

版本:
- Sencha Touch 2.2.0 beta

### 在Safari中调试.

Chrome中调试Ext.Anim, 涉及layout的不工作, left / top / weight / height, 在Safari / i-devices / Android Devices 上正常.

所以必须启用 Safari 进行调试. 

Chrome中尚无解决办法.

<!-- more -->

### Ext.Animator

这东西的确不存在, Component中用过他的代码都已经被override了, 不要耽误时间.

取代者是Ext.Anim

### 正确的Anim使用方法示例:

```js
	theAnim = new Ext.Anim({
		duration: 500,
        autoClear: true,
        easing: 'linear',
        to: {
            opacity   : 0.0,
        },
        from: {
            opacity   : 1.0,
        }
    });

    theAnim.run(this.elment, {
    	after: function(){
    		console.log('after');
    	}
    });
```

### 参考Anim.js 里 anims

这种写法避免了在每次需要用到anim的时候, 重新new anim对象, 避免内存泄露, 降低gc介入频率.
