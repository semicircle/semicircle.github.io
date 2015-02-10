---
layout: post
title: "Apache Thrift 爬坑行"
date: 2015-01-19 08:16:53 +0000
comments: true
categories: thrift protobuf iOS ruby 
---

####  什么是 Thrift

> 也不知道谁规定的, 当写一篇技术分享博客的时候, 第一个大标题必须是"什么是XXX". 

The Apache Thrift software framework, for scalable cross-language services development, combines a software stack with a code generation engine to build services that work efficiently and seamlessly between C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, OCaml and Delphi and other languages.

####  什么是爬坑行

就是趟应用一个新技术时遇到的各种坑. Common pitfalls 用英文的话.

####  时空座标

既然是 爬坑, 那就具有一定的攻击性, 所以, 锁定座标很重要.

**软件版本**

1. thrift 0.9.2
2. Mac OSX 10.9.X ~ 10.10.2

**时间区间**

2013.11.28 ~ 2015.2.3

---
*那么前戏结束, 爬坑正式开始*

# 1. 安装

没错, 坑从安装就开始了. 而且这个坑不小.

[THRIFT-2229](https://issues.apache.org/jira/browse/THRIFT-2229)

这个 Issue 说的是无法在 Mac 下编译 Thrift, 目前是 close 了, 但仍在困扰着一些人.

这里的 Issue 本身就是坑, 其实这些讨论更具误导性, 主要问题在于, 由于 Thrfit 这样的跨语言项目的复杂性, 其"编译安装"系统其实是有别于一般软件的, 大家都没有理解 Thrift 的架构, 而编译失败后, 又认定是这个项目不成熟.

我觉得这一点在相当大的程度上, **妨碍了大家对 Apache Thrift 的第一印象.**

> 要相信, Apache 也是在很努力的维护 Thrift 这个项目的! (大概8:20发

**首先, 你要理解 Thrift**

1. Thrift compiler 核心编译器: 将 thrift 接口定义文件, 编译成 Target 语言的源代码
2. Thrift libraries : 编译出的 Target 语言代码, 在 Traget 语言中执行的支持库
3. Target : 指的是 Thrift 支持的各种语言

**Thrfit 的编译系统**

它是这么工作的:

1. 首先编译用 c++ 写的 Thrift compiler
2. 查找是否具有各个语言的运行环境, 比如, 是否有 ruby 环境, 如果有, 则编译 ruby 版的 Library.

**Issue-2229 的实际情况是?**

1. 当年 10.9 的时候, Apple 用 clang 前端彻底替换了 gcc, 导致 compiler 无法被正确编译. 后来, Apache 奋力工作后, 到 `0.9.2`, 这个 bug 被修复了.
2. OSX 自带了 Ruby / Python / Java 等环境, Thrift编译系统发现系统中存在这些工具链, 就尝试编译了相关的 Library, 然后因为 Apple 自带的版本比较老旧, 编译就失败了.

**其实, 这些 Library 大部分是不需要自己编译的**

1. 用 `--without-java` 这样的编译选项, 略过相关 Library 的编译就可以获得 Success 的结果了.
2. 对于脚本语言, 直接使用 release 包里面的就可以了.
3. 有些是以源代码形式 release 的运行环境, 比如 Cocoa.
4. 有些库是以各种语言自带的包管理工具发布的, 比如 maven / gem / npm 等.

> 注意: 不要混淆 *库* 和 *编译器支持的语言*
> 比如: 即便用 Linux 环境编译的 Thrift, 其 Compiler 也是具有编译出 Cocoa 代码的能力的. (**是的, 就是这么跳脱!**)

**Protobuf 做得怎么样?**

Protobuf 面临相似的问题, 不过要简单许多: 只有3种语言, 运行环境没RPC.

Protobuf 不愧为 Google 主导的项目, 在 Mac 上编译, 测试, 不会有任何问题.

当然, 很多 Protobuf 的第三方语言支持, 做得都不怎么样. 比如, 之前我吐槽过的 iOS + Protobuf.

# 2. 编写接口文件

当在编写接口文件的时候, 会遇到第二个大坑:

> 前向声明在哪里?

以下代码会导致 Thrift compiler 报错:

``` 

struct UserInfo {
	1: optional string name,
	2: optional ProfileInfo profile,
}

struct ProfileInfo {
	1: optional list<BlogInfo> createdBlogs, // Compile error
}

struct BlogInfo {
	1: optional string title,
	2: optional string content,
}

```

`Compile error` 这行里, 会报"使用了未定义的 BlogInfo"错误, 因为, BlogInfo 的定义晚于 ProfileInfo 定义.

也就是说, 这里我们需要"前向声明"一下 BlogInfo. 那么, 问题来了:

> Thrift 不支持前向声明

在上述例子里, 我们只需要把 BlogInfo 放在 ProfileInfo 之前, 即可解决这个问题. 

不过, 如果这里的 BlogInfo 定义更复杂一点:

``` 
struct BlogInfo {
	1: optional string title,
	2: optional string content,
	3: optional UserInfo author,
}

```

这是, BlogInfo / UserInfo / ProfileInfo 三者之间相互引用, 构成了循环. 该怎么办?

> 恭喜您获得了 <<发现 Thrift 的死穴>> 成就.

不仅仅是 Thrift 的 Compiler 不支持, 目前 Compiler 生成的代码里, **有些**也是不能够被前向声明的. 举个例子: 如果是 C 语言, 那么在出现结构体嵌套时, 使用的是直接嵌套, 而不是指针. 这样, 即便 Thrift 的 Compiler 支持了前向声明, 那这种写法在 C 语言编译的时候, 也是非法的: 结构体不能直接嵌套. (*这里只栗子, C实现我没看过, 不过ObjC是如此的*)

这就意味着, 如果要添加这个特性, 连带静态语言的 Runtime Library 也都要大改一番了.

往好处设想:

> This is a feature, it's by design, not a bug. (post @ 8:20

也许是为了提升效率, 毕竟如果消息体很小, 从操作系统的内存管理机制来看, 反复 malloc 小块数据可能导致消息解析/组装性能严重下降. 如果想要提供这个特性, 则有必要在 Runtime Library 中添加配套的内存池技术, 才能获得出色的性能. 而就目前 Thrift 那些 Runtime 的现状而言, 能 compile 就谢天谢地了, 再添加这么复杂的特性属于强人所难. (等等! 我是不是说得太多了...

**Protobuf 做得怎么样?**

1. Protobuf 不需要前向声明, 可以递归嵌套.
2. 据第三方测试: Protobuf 的解析速度, 比 Thrift 更快.

> PB 完胜, 收工~

#### 没有前向声明的 Work around

在我的技术方案里, 规避方法是将同一个对象在表示时分层:

1. Base : 只包含该对象相关的 Thrift 基础类型, list<string> 之类也算基础类型.
2. Info : 包含该对象的 Base. 还有相关对象的 Base, 比如, BlogInfo 里 包含 `UserBase author`.
3. Detail : 类似 Info, 可以包含 Base 或 Info Level 的 struct. 与 Info 的区别是, Info 一般用于返回列表(多个对象)时使用, Detail 是获取单个对象时使用, 比如 getUserDetail时.
4. Result : 有的接口返回 `XyzDetail` 或 `list<XyzInfo>` 的 形式都无法满足需求, 那就专门创建一个对应该接口的 AbcdResult 结构.

应用了这套方法后, 基本满足了我的应用场景下的 API. 

> 我时常也为定义这样愚蠢的DTO而感到苦恼, 但是, 考虑到不需要自己实现通信模块, 还有那如丝般顺滑的调用体验, 想想心里还有点小激动呐~

# 3. Client编程

在使用 Thrift 生成的 RPC 代码的时候, 你会感受到:

> 经典 RPC 纯爷们儿, 就 TM 不向 RESTful 低头! 

- 众: 那你倒是支持异步啊, 
- Thrift: 有人说Callback-Hell众口难调啊.
- 众: 那咱支持个 Future, 没 Callback 了吧?
- Thrift: 整那些个模式逼格太高, 再说也不好用啊.
- 众: 那咱支持个设超时? 这个不过分吧???
- Thrift: 不需要, 咱们就是干白儿利落脆, 就是快
- 众: 那万一超时了怪谁? 阻塞了 UI, 用户体验不能丝般顺滑了该怪谁?
- Thrift: 那...那...那都是时辰的错!
- 时辰: ...对不起

(*其实我只用了 Cocoa 的Client Library, 大概并不客观*)

#### 没有异步的iOS Work around

首先定义一个 block 类型:

``` objc
typedef void(^UIUpdatingBlock)();
```

创建一个 Utils 类, 在其中增加静态方法: (生成的 service object 名为 AccountServiceClient )

``` objc
+ (void)invokeAccountService:(InvokingAccountServiceBlock)block;
{
    static dispatch_once_t onceToken;
    static dispatch_queue_t _thriftRpcQueue;
    dispatch_once(&onceToken, ^{
        _thriftRpcQueue = dispatch_queue_create("com.myapp.thriftq", NULL);
    });
    
    dispatch_async(_thriftRpcQueue, ^{
        THTTPClient * transport;
        TBinaryProtocol * protocol;
        AccountServiceClient * client;
        
        @try {
            transport = [[THTTPClient alloc] initWithURL: [NSURL URLWithString:@"http://server.url/"]];
            protocol = [[TBinaryProtocol alloc] initWithTransport:transport strictRead:YES strictWrite:YES];
            
            /* get the service object */
            client = [[AccountServiceClient alloc] initWithProtocol:protocol];
            
            UIUpdatingBlock b2 = block(client);
            dispatch_async(dispatch_get_main_queue(), b2);
        }
        @catch (NSException *ex)
        {
            NSLog(@"connection problem: %@", ex);
        }
        @finally
        {
            client = nil;
            protocol = nil;
            transport = nil;
        }
    });
}
```

这样在调用的地方: (调用的 getUserDetail)

``` objc
	[CHXUtils invokeAccountService:^UIUpdatingBlock(AccountServiceClient *client) {

		// 调用 service object, 如果需要 Catch Exception, 也在这里. 
        self.user = [client getUserDetail:_userId];
        
        return ^void(void) {
            // 更新 UI 的地方
            // ...
        };
    }];
```

我相信更有经验的 ObjC 选手会利用 *protocol* 技术创建更合理的设计, 我这里只是做了一个必要的封装:　

> 毕竟, 我们不能在 Main Thread 里调用 service object 的方法, 这会阻塞 UI 的.

(写到这里, 我才意识到, 我从来没试过在 Main Thread 里调用 service obejct, 我猜 iOS 会狂飙异常吧)

*2015.2.9补充:*

1. (经别人提醒想到)在更新 UI 时, 操作的指针应该是设置了 `__weak` 属性的, 否则, 在调用过程中如果 UI 已被退出了, 这里的 block 引用将导致其无法正确释放内存.
2. dispatch_once 的部分还是在 UI Thread 上, 分配内存就有可能导致卡顿, 应该移动到其他位置.

# 4. 性能

由于, 我并没有机会公平的测试当 Thrift 流量大了以后, 服务器性能的占用情况.(*这里无法公平的原因是, 我自己用 go 语言配 Protobuf, 用 ruby 配 Thrift, 所以没有绝对公平的比较环境*)

但是, 在使用 Thrift 的过程中: 

> 用 Wireshark 发现, 每次请求似乎并非一次通信

这是一种奇怪的协议实现, 出现在用 Ruby 的 Runtime Library + TCP Transport 环境下. 

如果你对性能优化敏感的话, 你大概已经意识到这里的问题了.

如果是 HTTP Transport, 倒是明确的一次 Request, 一次 Response. 但问题是, 真正在内部网络下, 追求性能时, 大概应该是 TCP 大显神通的地方, 现在却..

其实, 这一点还是属于存疑状态, 毕竟是号称高性能的RPC. **也许是我的打开方式不对.**

# 综上

Apache Thrift 作为在 Facebook 实际使用的 RPC 系统, 其可用性还是值得肯定的, 抛去对小白用户不友好, 和没有前向声明的两个大坑之外, 其他的小坑对生产力的破坏并不明显, 总体而言, 如果你不是 RESTful 铁杆粉, 而且想要给 Client 的程序员提供一个相对舒适的编程环境的话, Thrift 还是你值得信赖的选择.

同时, 如果你的业务逻辑相对复杂, 特别是对象之间关系相对复杂时, 使用 Protobuf 会节省你在 Data Transfer Object 定义上所消耗的时间和精力, 相应的, 你要付出一部分实现消息通信的精力, 换取比 Thrift 更好的性能.

* Thrift: 对小白用户不友好, 有局限性, 省事.
* Protobuf: 性能更佳, 可扩展性强, 必须要自己写通信模块.

请自取所需.
