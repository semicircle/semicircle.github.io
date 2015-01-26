---
layout: post
title: "Best practise of Sencha Touch mutil-controller communication"
date: 2013-05-21 17:51
comments: true
categories: sencha html5
---

I get the solution from several related pages.

### Events are better than direct-call

A controller can be get by `this.getApplication().getController()` and call the functions directly, but by this way, the controllers may get coupled too tightly. 

Sometimes, you want the communcation's result back immediately, in that situation, you may consider to merge the two controllers into one. (This immediately is not about synchronize between threads, it's about code's organization. Because in most case, fireEvents means to call the handlers in the current stack.)

### Global events

In sencha doc's guide part, Obviously, there are no such thing documented.

To achieve this, code like below:

```js
this.getApplication().on('MyGlobalEvent', function(){});

this.getApplication().fireEvent('MyGlobalEvent', ...);
```

