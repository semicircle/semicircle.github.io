---
layout: post
title: "基于 Docker 的(轮子组装)开发环境(有待完善)"
date: 2015-01-19 07:53:25 +0000
comments: true
categories: docker
---

# 前因

为什么要写 Docker

### 近况

首先, 从这大半年的时间没有更新博客说起, 因为之前不小心破坏了 gem / bundle 的环境, octopress 全线罢工. 又由于一直没有用 ruby / rails 开发的任务, 也就没有尝试恢复. 

某次从 Twitter 上看到一句被广为转播的话, 大意是: If you can't setup your develop environment in a single command, then you're working in a prototype. 那一次, 又感到自己土鳖了...

### Docker 的意外收获

之前, 由于一直使用 go 编写服务器端, 所以关注到了 Docker 这个 go 社区里的明星, 并稍微学习了一下. 没想到后来意外收获让我始料不及.

这个意外收获就是 Docker Image Repo. 利用公开的 Docker 现成的 Image, 玩了很多轮子:

- Shadowsocks, 我最后用的也就是一个别人的 Image.
- Postgres,  用的官方的Image, 据说安装 Prostgres 是非常蛋疼的一件事.
- Openstreemap 我自己安装两三天未果, docker image 下载下来即插即用.
- Apache Solr 一般要先安装 Java.
- Gitlab 谁装谁知道这玩意多 tm 难伺候!!
	
同时, 借助 Docker 的功能, 我还创建了一些工作环境:
- Protobuf编译, 
- Thrift编译, Mac下一直编译不过, 用一个干净的
- Rails
- 本 Blog 的 Jeklly 环境.

这些使用 Docker 的应用过程值得总结一下.

### VagrantUp 的不足之处

首先要承认我不是资深的 ruby 程序员, 所以对 ruby 整套软件栈并不是太熟悉. 缺少一些属于 ruby 社区的 common sense.

但是, 我觉得我的批判应该还是准确的, 因为我所批评的, 都还是一些 "基础" / "入门" 的环境配置过程中产生和发现的问题:

###### 环境不兼容问题

最初我想配置 Vagrantup + Rails 环境时, 在 google 中搜索 `vagrant rails` 的结果, 按照出现在第一位的教程走一遍, 结果遇到莫名其妙的错误, 在探索了一段时间以后, 找到是 rbenv / chef 的兼容问题. [具体情况见我的评论信息](https://gorails.com/guides/using-vagrant-for-rails-development#comment-1759266752) 此外, 这个教程讲述的方法里, 还有 mysql 版本的问题, 看评论是半年前就已经存在的问题了.

###### 很多看起来区别不大的轮子

这里面说的其实是 chef / chef_cookbook / chef_solo 等等许多用来 "发布" / "监控" / "自动化配置" (provisioning) / "集群管理" 的工具. 我花了很长时间, 弄不清这些东西的区别, 及他们各自解决的问题. 我100%的认可他们各自是有各自的用途的, 而且对于"专业"的ruby/devop工程师, 应该是清楚他们的作用的, 但对于入门, 只是想用 Vagrant 搭建dev环境的人而言, 这些东西显得太重了.

###### 到处都是脚本

脚本就是程序, 程序就是会升级, 升级就是会出现不兼容(特别 ruby 还是如此动态灵活的语言).

###### 不就是想用 VagrantUp 搭个 dev 环境吗? 扯那么多干啥.

问题就在于此, 如果你不明白 chef cookbook solo 等一大堆名词, 你就无法愉快的用 VagrantUp 玩耍, 比如你想在 dev 环境安装 postgresql, 你就要懂得 solo 什么的脚本, 否则, 一个 vagrant destroy命令, 你搭建的环境就灰飞烟灭了, 你无法用 vagrant up 一个命令重建起来, 那么你还是在一个原型里编程, 一切都还是那么low. 

# 囫囵吞枣式 Docker 简介

论正确的使用姿势:

- 如果你是初次接触 Docker, 这里的简介过于笼统. 如果读到后面都不下去, 完全正常, 不是你的智商问题.
- 如果你已经熟悉了 Docker 的一般操作, 前半部分可能稍显枯燥, 后半部分有我的心得, 说不定你能有所收获. (还有可能你会觉得我是个sb..)
- 正确的姿势是, 请把这些内容当小说.

### 集装箱的类比

在远洋运输业出现集装箱以前, 货物在港口装船的码放与装卸是非常令人头痛的问题. 要考虑到货物的形状, 重量与船的平衡, 多个港口的到达顺序等等. 往往装船过程需要很多时间, 占用码头不说, 货物也会有很多损耗. 在集装箱出现以后, 各种奇形怪状的货物都被要求装入集装箱, 这让港口的装卸货效率得到大幅提升, 将航运业的生产效率提升了一个档次.

Docker 就是希望成为服务器软件航运业(软件集成+部署)的集装箱, 任何服务与软件都装入 Docker 提供的 Container 中, 包括软件本身和他所依赖的运行环境, 只暴露出对外服务的网络端口, 以此来实现生产效率的提高.

### Dockerfile / Image / Container

Dockerfile / Image / Container 是构成 Docker 技术的基本元素: 

###### 什么是?
Dockerfile 用来创建 Image, 通过命令行 docker run 来启动Image, 形成Container.

Dockerfile是创建Image的脚本:

``` Dockerfile
FROM ubuntu
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y x11vnc xvfb firefox
```

通过 docker build . , 可以运行Dockerfile脚本, 创建一个Image. 通过 docker images 命令查询.

通过 docker run, 可以把  image 运行起来, 一旦运行起来, 就被称为 container 了.
典型的命令: docker run -i -t ubuntu bash,  以bash为CMD, sh -c为Entrypoint, 产生一个container (集装箱). 接下来的命令行都是运行在集装箱里的.
container 可以通过commit, 转换称为image?

通过 docker ps 查询系统内正在运行的 container,  
docker ps -a 查询所有的, 包括结束的 -> 可以被commit成image.

###### 为什么?

要理解为什么会有Dockerfile / Image / Container的设计, 可以按照这个思路想: 

1. 先有的Container: docker最主要的功能, 将代码或程序装在集装箱(container)里 ship. 在鲸鱼号集装箱船服务器上,  程序在container里折腾, 保证对内部程序是面向container, 外部服务器也看到container, 从而实现沙箱隔离, 程序的运行环境也单纯下来.  在这个层次上, 不使用Dockerfile, 甚至, 从来不commit中间步骤, 也无所谓, 只是为了一个集装箱.

2. 后有的Image: 如果每次升级, 都手动从ubuntu创建image,  那每次输入若干软件与环境相关的命令, 就违背了DRY原则,  所以就有了 image / container 的 commit动作, 通过命令行, 搭建好一次环境就OK了, 不需要反复敲apt-get安装软件. 有了image, 每次升级代码, 把这个image run起来,  形成 container, 把内容程序或代码替换了, 再commit成image, 把image ship到服务器, 在服务器上run起来为 container, 就ok了.

3. 最后又有了Dockerfile: 前面提到了安装好环境的image, 这个环境是固定在某个特定版本上的, 但环境软件, 也是会升级的, 要定期重复apt-get那一套? Dockerfile就是解决这个问题的, 通过build Dockerfile, 所有命令都重新执行了一遍.

4. 全能选手Dockerfile: 除了3提到的场景, 其他任何会执行重复命令的场景, 都可以用Dockerfile来实现自动化.

到这里, 囫囵吞枣式简介就已经完成了.

# 我是如何搭建环境的

### 纯手工搭建一套运行环境

任何想依赖一套脚本达到运行环境的, 比起从shell敲命令的初学者方式, 都是要费更多时间的. 所以, 通过 `docker run -it ubuntu bash` 为起点, 你可以在15分钟内搭建起任何一种主流开发技术的初学者环境. 

而如果使用脚本, 比如 cookbook / Dockerfile 之类的技术, 你会耽误更多时间在调试(学习)脚本上.

创建完环境和每次修改后, 别忘了 commit 环境到 image.


### 每次启动你的环境

请创建一个一行脚本专门用来启动各个环境

``` sh
docker run -it -v $PWD/workspace:/workspace -p 3000:3000 --link some_postgres:postgres --link sunspot_solr:solr my_repo:5000/rails bash
```

解释:

1. -v $PWD/workspace:/workspace 这是将当前路径下 workspace mount到了新启动的 container 的 workspace 下
2. -p 3000:3000 这是 rails 的服务端口
3. --link some_postgres:postgres 我使用了 postgresql 服务, 这里需要另起一个 postgres 的 image / container, 直接是官方的镜像.
4. --link sunspot_solr:solr 我使用了 solr 服务, 不是官方的, 但也是 Public Repo 里直接下载的.
5. my_repo:5000/rails 这里 my_repo 就是我自己的 private registry. 下面会专门讲到这里.

### 备份/分享你的环境

根据docker标准的用法, index服务和registry服务, 都是公共的.
但现实是GFW的存在, 导致几乎完全无法正常访问registry. 所以必须创建private registry.
而private registry的位置又成了问题, 因为从墙外访问墙内的服务器, 也会不稳定.
要是使用国内的第三方服务, 被爆菊真的没关系吗?
所以, 最终的规避方法是: 在本地创建private registry, 其他服务器通过ssh反向隧道来连接到本地的private registry. 而private registry本身是最简单的版本:  直接pull 一个 samalba/docker-registry 下来, 先用着...

``` sh
$ docker pull samalba/docker-registry
$ docker run -d -p 5000:5000 samalba/docker-registry
```

其实这里还有一个问题, 就是早期的 Docker 并不强制使用 https, 而是使用http下载, 这其实是存在安全隐患的, 这里使用反响隧道, 也保证了安全性.

### 特别的注意事项: 绑定端口

很多本地调试的服务器程序, 比如 rails / octopress / jekyll, 他们 bind 的 host, 都可能是 127.0.0.1, 在 Docker 的 这种配置下, 从 Host machine 都会出现无法打开的问题, 需要特别的指出 0.0.0.0, 比如 `rails server -b 0.0.0.0`.

###　(此坑待填)

1. 关于 Data volume 的正确理解与使用
2. 在 Mac 使用 Boot2docker 的注意事项
3. 如何处理敏感数据, 比如 ssh private keys.
4. 哦哦哦~ 社会是~ 伤害的比赛~

# 结束

本篇只是介绍用 Docker 搭建开发环境的, 其实 Docker 的真正用途岂止于此. 
我自己的技术栈里, Docker 基于了我更多的帮助, 比如快速集成各种需要编译/配置等等很多繁琐步骤才能搞定的服务, 

这篇文章里的很多部分都是源自这大半年的工作笔记, 一直感觉必须要写点什么, 否则亏欠 Docker 良多啊..
纵观我自己的笔记, 还有许多关于 Docker 的"奇技淫巧", 和其他篇的内容有所重叠, 暂时还没构思好如何呈现, 以后也许有机会再更新.
