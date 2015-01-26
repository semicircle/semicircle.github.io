---
layout: post
title: "sencha touch 的学习曲线"
date: 2013-04-09 10:51
comments: true
categories: html5 sencha
---

什么是Sencha Touch? 什么学习曲线?

Sencha Touch是一套用HTML5技术开发移动应用程序的架构, 与jQuery Mobile的类似. 

相比较之下,
Sencha Touch的特点是: 1. 开发出的App性能/流畅度能够媲美Native App, 2. 对Android的兼容性好, 3. Sencha Touch用JS创建DOM, 开发过程也主要使用JS

关于Sencha Touch和jQuery Mobile的比较, 请参见[这篇](http://css.dzone.com/articles/sencha-touch-v-jquery-mobile), 或自行google.

众所周知, 作为一种技术方案, Sencha Touch的学习曲线是比较陡峭的, 
但为了追求跨平台的特性, UI的高流畅度, 想想铤而走险也值得了.现在...作为一只尝试Sencha Touch三周有余的小白鼠, 我觉得有必要把经历过的各种"坑"都总结一下,以儆效尤...
<!-- more -->
首先, 要说明一下, 作者在UI开发方面, 有如下背景: MS家的VC/WinForm/WPF, Android Native, 一些粗浅的HTML/CSS/Javascript经验. 列出主要是提示: 如果你的背景不同, 也许感受会截然不同.

其次, 本着明枪易躲暗箭难防, 这里主要写写"暗箭", 即你意想不到的困难, 在明面上的, 就不赘述了.

正文开始:

## 如果你从Android/iOS Native App的世界来
   这意味着, 你已经熟悉了Native App开发, 或者更宽泛的说, 熟悉了,任何使用XML或资源文件定义界面, 然后用某种静态语言进行逻辑开发方式.	

### 只需要学习Javascript? 
    "学习新语言有什么难的, 写博客还要新学习Markdown呢~"
实际上不止一种新语言,

在大部分Sencha Touch的视频里, 往往只能看到讲演者编写JS的部分, 在实际应用中, 调试过程中,或从理解原理的角度, 是一定需要扎实的掌握HTML和CSS的知识的.

此外, Sencha Touch集成了Compass, Sencha工程的样式文件都是通过Sass编译而成的, 很多界面组件都开放了一些Sass变量用于调整外观, 所以Sass也变成了必学.

### 恼人的拼写错误, 没有检查机制.
在静态语言的开发环境里, 以Android为例, 当你在layout的xml文件里, 拼写错了一个element的名称或属性后, 是不可能通过编译的. 
而在Sencha Touch的环境里, 假设:当你extend一个Container, 想要创建自己的界面, 通过items属性定义当前Container的内容时, 如果不小心把items拼写错了, 那么:

- items不是一个Container所必须具备的属性;
- 同时, JS的动态语言特性, 也允许你在这时给对象任意添加属性, 比如"itme", "ietm"都是合法的

所以悲剧就这样发生了, 没有任何环节会提示给你任何错误信息, 只是: 你定义的Items, 不再生效了, 你的Container里没有内容, List里空无一物, Form里没有任何Field … 当你若干时间以后, 你发现这个错误以后, 除了捶胸顿足, 恐怕还会抱怨:　拼写错误Sencha怎么也不报错啊??　(在微博搜Sencha,　十有八九会见到这样的吐槽贴).

### 思维方式难以适应: 
如果你已经在Java/Cpp等环境适应了OO的思路, 到了Sencha所基于的ExtJs这里, 事情开始变得不同.

这里有个例子, ExtJs Class System的一个类具有configs, 可以理解为: configs是在类实例化时传入的属性表或参数表, 实例化之后, ExtJs自动在对象上为每个属性生成setter/getter, 同时用户可以"重载": applyXXX, updateXXX方法(XXX为具体某个属性名),  这样在configs被应用在对象上时会调用applyXXX, 在发生变化时调用updateXXX. 这些其实都还好理解, 但接下来: apply函数的返回值会顶替setter传入的值, 成为保存在属性上的值, 而且这个顶替者, 可以是与setter传入的是不同类型的, 这也就是说, 可能你调用某个属性的setter设置了一个String, 再调用相应的getter, 可能给你返回一个Component, …这种情况, 非常普遍, 

更令人发指的是, 有些文档中, 比如DataView, 他给出的Demo和Best Practise, 就是让你[自己写一个这样applier](http://docs.sencha.com/touch/2-1/#!/guide/dataview-section-4)
    
#### 我猜测ExtJs会这么设计的原因 

- 根据架构从实践而来的原理: JavaScript是一种弱类型语言, 变量可以被先后赋予不同类型的值, 在长期的实践中, 奇葩成自然, 走成了路, 这样的设计也就进了架构.
- 另外, 实事求是, 这样的设计减少了编码量, 假设我要初始化一个表单里DatePickerField, 它本身包括一个DatePicker的Component, 创建时, 我只要传入一个js的map给picker这项config即完成了对DatePicker的配置, 要是静态语言, 一般的思路是, 我得拿着这堆参数先创建一个DatePicker, 再拿着这个DatePicker再去创建DatePickerField.
    
除此之外, 比如, 实例可以通过override覆盖方法和配置, 等等, 不胜一一吐槽了.

### 要掌握Chrome的调试环境.
调试JavaScript的工具, 也许还有Firefox的Firebug, 无论如何, 需要掌握一个这样的环境,
这点貌似显而易见, 但我想说的是, 想熟练掌握也需要时间, 

关于这里的技巧, 回头另开文章介绍.

## 如果你从HTML/CSS/JavaScript的世界来,
具有jQuery等开发经验, 笔者不是这种背景, 只是妄自揣测下:

- ExtJs的Class System, 也许和其他JS库的思路不同, 需要适应.
- 如果要做Native程序, 可能需要熟悉各种native程序的打包方式, PhoneGap等.

如果这部分帮助不大, 请看下一部分.

## 无论你从哪里来
### 文档欠缺:
1. 不要想像有 MSDN / Android / iOS那样完善和全面的文档, 
2. Sencha的文档看似很多, 但实际大部分内容类似注释, 信息量很少. 很多新特性尤其如此.
3. 遇到问题基本靠猜, 靠源代码. (也好在有源代码~)
     
### Sencha Touch的出错提示较少
与拼写错误不能被检查到类似, 但有区别

举一个实际例子, 如果一个Container没有定义layout属性, 而且包含了Toolbar, 那么可能出现内部的DOM被创建, 但界面上肉眼无法看到.
遇到这个问题时,  修改css的z-index之类也不生效, 其实可能是通过transform之类的高端css特性隐藏,

这非常接近于用Sencha Cmd创建出来的原始工程, 很多新手都遇到这样的问题: [请看](http://stackoverflow.com/questions/9685062/how-do-i-make-a-dataview-list-visible-in-sencha-touch-2)

### Android的碎片化问题仍需面对:
用HTML5开发也躲不过. 虽然Sencha已经对很多问题进行了规避, 但仍有很多琐碎的细节问题, 

比如, 创建Ext.Date时, 如果传入的String没有时区信息, 在2.3上认为传入的是本地时间, 而4.0上则认为传入的是标准时间
    
### HTML5分裂问题:
一定会遇到的问题, 全靠 caniuse.com / phonegap

### 运行环境与调试环境总是有差异的.
Chrome在mac/pc上调试, 但运行环境的browser总与调试环境不同, 特别是, 运行环境里用到的PhoneGap也是调试环境里没有的.

在调试环境上, 你能依赖的只有alert和console.log这样的原始手段.
