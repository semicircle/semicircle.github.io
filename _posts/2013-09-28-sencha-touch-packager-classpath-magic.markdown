---
layout: post
title: "Sencha Touch Packager的classpath黑魔法"
date: 2013-09-28 18:20
comments: true
categories: sencha
---

今天升级了 Sencha Touch到 2.2.1, Sencha Cmd到 3.1.2.342,
创建新工程后, 发现一个诡异的问题.

### 应该是这样的:

以往, 在工程根路径下, 我习惯把通用的组件或逻辑写在独立的ExtJs Class里, 不和app的model/view/controller混在一起.

大概的路径如下:

```
+ appbase
++ app
+++ model
+++ view
+++ controller
+++ store
+++ ...
++ Ux  <--自定义的组件位置
+++ foo
++++ bar.js
```

同时, 在app.js里, 为了支持Ux:

```js app.js
Ext.Loader.setPath({
    'Ext': 'touch/src',
    'MyApp': 'app',
    'Ux': 'Ux'
});
```

就可以按这样的形式引用:

```js
	requires: [
		...
        'Ux.foo.Bar',
        'Ux.foo.Bar2',
        ...

    ],
```

就好像Ux和ExtJs位于不同的Namespace的感觉, 很大气上档次吧~

### 可现在的实际情况是:
<!-- more -->
在升级了 Sencha Cmd 和 Sencha Touch之后, Micro Loader可以正常的运转, 也就是说在浏览器Debug时, 一切都是正常的.

但在打包时, `sencha app build package`, 糟糕的事情发生了:

```
[ERR] C2008: Requirement had no matching files (Chat.Api) -- /Users/semicircle/goplace/src/bitbucket.org/semicircle/chatservice/demo/www/app.js:22102
[ERR] The following error occurred while executing this line:
/Users/semicircle/goplace/src/bitbucket.org/semicircle/chatservice/demo/www/.sencha/app/build-impl.xml:256: The following error occurred while executing this line:
/Users/semicircle/goplace/src/bitbucket.org/semicircle/chatservice/demo/www/.sencha/app/build-impl.xml:249: com.sencha.exceptions.ExScript: Wrapped com.sencha.exceptions.ExBuild: Failed to find any files for /Users/semicircle/goplace/src/bitbucket.org/semicircle/chatservice/demo/www/app.js::ClassRequire::Chat.Api (x-app-build#291)
   runAppBuild (x-app-build:291)
   [anonymous] (x-app-build:571)
   x_app_build (x-app-build:569)
   <script> (anonymous:1)
```

这里的 `Chat.Api` 就是我想独立出来的模块.


### 解决方法是这样的:


经过长时间阅读Sencha Cmd的文档, 都没有找到答案, 后来, 无意之间发现了这个: `.sencha/app/sencha.cfg`

```cfg .sencha/app/sencha.cfg
   2 ...
   3 app.framework=touch
   4 
   5 app.classpath=${app.dir}/app.js,${app.dir}/app,
   6
   7 app.build.dir=${workspace.build.dir}/${app.name}
   8 ...
```

将这里的 `app.classpath `增加 `${app.dir}/Ux` , 即 `app.classpath=${app.dir}/app.js,${app.dir}/app,${app.dir}/Ux` 即可实现对 Ux的引用.

这里的变化完全没有记录在任何文档中, 我以前的想法里, Sencha Cmd是同u哦 app.js 里 Ext.Loader的代码来检测到底要如何找ClassPath, 但目前这个机制好像发生了变化, 据(SenchaCmd文档)说这里Ext.Loader不填写, 也能正确加载. 添加新特性的同时, 对Ux的加载被忽略了, 在目前的Sencha版本里, 这个classpath设置成了名副其实的黑魔法, 希望未来的版本能修正这些问题.



















