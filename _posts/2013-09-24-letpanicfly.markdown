---
layout: post
title: "让panic再飞一会儿"
date: 2013-09-24 12:36
comments: true
categories: golang
---



有go黑说: "在Go程序里的`if err != nil`太多了"

不可否认, 这确实是很多Go语言的程序的面貌, 难道Go本身没有异常处理机制么?
其实不是, Go语言提供了panic/recover机制, 简要介绍一下:

### Go的异常处理机制

```go

func a() {
	if something_wrong {
		// 让程序进入panicking状态
		panic("something wrong")
	}
}

func b() {
	a()

	defer func(){
		if x := recover(); x != nil {
			//处理panic, 让程序从panicking状态恢复的机会
		}
	}()
}

```



解释: 当调用panic以后, 程序就进入了panicking的状态, 此时, 程序会沿调用堆栈一路向上返回, 同时, 途径的defer都会被调用到, 直到遇到某个defer的函数X中的recover, panicking状态截止, 得到了处理panic的机会. 从这个X退出后, 堆栈会沿着调用defer X的程序的上一级 继续运行.
<!-- more -->
如果从java的异常机制相比较: try / catch的模型 , panic就相当于throw了一个exception, 遇到recover就相当于遇到了catch, 途径其他defer相当于finally里释放资源的代码.

不同的是, go缺少 try block, 一个有recover 的 defer就罩住了整个函数里的 异常状态. 

另外, go默认不限制 panicking不断的往堆栈上方飘, 而java必须要用throws关键字, 否则编译不通过. go没有在编译时控制这种行为的方法. 

个人认为, go的异常处理模型, 第一眼看来, 比java稍微难理解一点, 但实际用起来, 由于不需要try / catch 这样的大block, 倒也让代码清净不少, 更重要的是, 写代码时更容易进入到心中无error而handle error于无形的境界, 让你能更专注代码逻辑.

So, don't panic

那么:

### 为什么Go语言的代码到处是 `err != nil` 呢?

我想有两个原因:

#### 1. 这其实是一种优化的设计

由于java不存在 error这样的类型, 或者说是函数只能有一个返回值, 不能夹带error这样的玩意, 导致错误必须由 exception返回. 而众所周知, java的exception比普通的函数返回大概多30倍的开销(其实是道听途说: [google-nuts](https://groups.google.com/forum/#!searchin/golang-nuts/panic$20recover/golang-nuts/HOXNBQu5c-Q/Je0qo1hbxIsJ)), 这样会让 库 的编写者难堪, 因为库的编写者不知道一个调用在应用场景里, 失败的概率是多少, 而java规定只能通过异常返回. 比如有时, 调用者只是通过调用文件读取接口, 确认一下文件是否已经读完了, 即他预期的就是一个失败的结果, 在这种情况下, 他必须用try/catch包围, 并且承担额外的运行时开销.

而go的库开发者, 可以从容的选择通过error来返回错误状态, 他们根本不用管这个调用是预期成功的还是失败, 用户自然可以通过error来判断. 没有多余开销, 没有大的try/catch block. (当然, 很多 `err != nil`)

#### 2. 历史原因

由于go的异常处理是在发展过程中引入的, 而且官方的库都没有用这套panic/recover机制, 所以有人不熟悉, 另外, 可能有人并没有领会库作者 总是 返回 error 的意图, 借鉴过各种库代码的风格后,  在自己代码里, 也经常用 error作为返回值. 当代码变得复杂后, 多个模块之间, 多个层次之间, 总是以error返回值处理错误流程, 逐渐的越来越多的精力被放在了 `err != nil` 上.., 对此, 我们的策略是:


![42]({{ site.url }}/images/letpanicfly/42.png)

### 让panic飞一会儿

在自己的应用程序里, 由于你可以确切的知道每个调用所预期的结果, 所以, 让失败结果panic起来, 就成为一件正常的事.

我目前采取的原则是:

- 在任何外部库函数调用处, 如果返回值"不合我心意, 又没有挽回的余地", 则立即让程序panic起来.

- 在自己的代码内部, 模块与模块之间, 全面禁用error作为返回值

- 让panic飞一会儿: 允许panic穿越多层调用, 在合适的位置, 统一recover, 比如, 某次服务请求, 如果有任何底层细节逻辑导致panic, 我会在这次请求的入口处defer来recover这次服务请求的panic问题.


最后的杀手锏, 一些这样的helper function, 帮我缩减了大约30%的代码行数: (<--什么是高端黑)

``` go
/* some helper functions */
func trueOrPanic(exp bool, what interface{}) {
	if exp == false {
		panic(what)
	}
}

func falseOrPanic(exp bool, what interface{}) {
	if exp == true {
		panic(what)
	}
}

func okOrPanic1(err error) {
	if err != nil {
		panic(err)
	}
}

func okOrPanic2(dontcare interface{}, err error) (interface{}, error) {
	if err != nil {
		panic(err)
	}
	return dontcare, nil
}
```


