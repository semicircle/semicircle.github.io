---
layout: post
title:  "逃亡: 从 Octopress 迁移到 Jekyll"
date:   2015-01-25 17:31:42
categories: jekyll update octopress jekyll
---

因为 Octopress 实在是没法用了, 所以正在迁移到金克拉...哦,不: Jekyll. 

实属无奈,战略逃亡.

图片坏了, 也许是件好事, 毕竟是写字的地方, 又不是摄影博客.

代码高亮坏了, 这可不是好事, 怎么办呢...

Discussion 坏了, 没有关系, 我就没见过我的博客有人留言~.

# 代码高亮方法:

想要调整为 Github 式的 高亮, 需要 在`_config.yml`里增加:

```
markdown: redcarpet
```

并且, 不要忘了

``` sh
gem install redcarpet
```

[爆栈网链接](http://stackoverflow.com/questions/13464590/github-flavored-markdown-and-pygments-highlighting-in-jekyll)

# 永久链接:

以前留给别人的链接, 说好要做彼此的天使呢:

还是修改 `_config.yml`

```
permalink: /blog/:year/:month/:day/:title/
```

# Disqus 

[这篇写得很细致](http://joshualande.com/jekyll-github-pages-poole/)

至于我这里, 实在懒得加了, 反正也没留言, 哈哈哈.


# 感谢

[逃亡者带路人](http://www.campaul.net/blog/2014/05/08/moving-from-octopress-to-jekyll/)
