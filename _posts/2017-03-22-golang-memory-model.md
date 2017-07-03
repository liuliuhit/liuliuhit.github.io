---
layout: post
title: golang memory model
---

[The Go Memory Model](https://golang.org/ref/mem)的翻译和学习笔记

<!--more-->

## Introduction
golang的内存模型详述了"在goroutine中对于变量的读操作能够监测到其他goroutine中对该变量的写操作"的条件.

## Advice
当程序中多个goroutinue并发的修改某个变量的时候必须对这些访问操作序列化.
为了序列化这种访问操作,可以采用`channel`或者使用`sync`,`sync/atomic`包中的原生的同步(**synchronization**)来保护共享的数据.

如果你一定要阅读这篇文档的剩余部分,以理解你程序的行为,you are being too clever.
Don't be clever.

## Happens Before, 参见[Happened-before](https://en.wikipedia.org/wiki/Happened-before)

在单个goroutinue中,读和写操作执行的行为要和在代码中描述的一致.编译器和处理器会对读和写操作的执行进行重排(指令重排),但仅限于不会对执行行为和结果与代码描述相违背的情况下才会进行这种重排.由于这种重排,两个不同的goroutinue会对相同变量操作顺序的感知不一致.举个🌰 ,在一个goroutinue中执行了`a = 1; b = 2`,在另一个goroutinue中监测到的可能是b的更新先于a.

为了具体描述这个读和写的问题,我们定义了**happens before**,描述go程序中对内存操作执行的顺序.如果事件e1 happens before事件e2,那么e2 happens after e1.如果e1既不happens before e2,e2也不happens before e1,则我们称e1和e2是happen concurrently(并发)的.
在单个goroutinue中,happens-before的顺序和代码描述的顺序是一致的.

下面定义读操作r和写操作w和变量v,读操作r能够观测到写操作w对变量v的修改,需要满足以下两个条件:
1. r不发生在w之前.
2. 在w发生之后,r发生之前,没有其他的写操作w'. 

为了保证读操作r能够观测到某一个特定的写操作w对变量v的修改,需要确保写操作w是唯一的写事件,除此之外还需满足下面两个条件:
1. w发生在r之前.
2. 其他的写操作发生在w之前或者r之后.

第二组条件比第一组更加严格,要求没有其他的写操作与w或者r操作并发.

在单个goroutinue中,由于没有并发的存在,上述两组条件是等价的,读操作r可以观测到最近的一次写操作w对变量v的修改.当多个goroutinue访问一个共享的变量是,则必须使用同步机制来建立happens before的条件,确保读操作r能正确的观测并读取到写操作w的修改.

变量的初始化也是属于这个内存模型.
读取超过一个机器字长的的数据,是无法保证操作顺序的.

## Synchronization

### Initialization
程序的初始化是在单个goroutinue中执行的,但是goroutiune可能会创建其他的并发的goroutinue.

如果包p引用了包q,q的init函数完毕happens before包p的任何一个函数.
main包中的main函数,happens after所有的init函数执行完毕.

### Goroutine creation
使用go语句,启动一个goroutinue发生在这个goroutinue的运行之前.

{% highlight golang %}

var a string

func f() {
	print(a)
}

func hello() {
	a = "hello, world"
	go f()
}

{% endhighlight %}

调用`hello()`会在未来某个时间点打印*hello world*(可能发生在`hello()`函数返回之后).

### Goroutine destruction
某个goroutinue的退出不会确保发生在任何事件之前,举个🌰

{% highlight golang %}

var a string

func hello() {
	go func() { a = "hello" }()
	print(a)
}

{% endhighlight %}

a的赋值操作没有遵循任何的同步事件,所以没办法保证这个操作会被其他goroutinue观测到.实际上一个激进的编译器可能会删除这个整个的go语句.

如果某个goroutinue的执行结果必须被其他goroutinue观测到,可以使用同步机制,比如lock或者channel通信来建立一个相对的顺序.

### Channel communication
channel通信是goroutinue间同步的主要方式,每个goroutinue在一个特定的channel上的发送操作,相应的通常会有另一个goroutinue在这个channel上接收数据.

**发送操作发生在相应接收操作完成之前.**

{% highlight golang %}

var c = make(chan int, 10)
var a string

func f() {
	a = "hello, world"
	c <- 0
}

func main() {
	go f()
	<-c
	print(a)
}

{% endhighlight %}

上面的🌰 可以确保打印出"hello, world",a的写操作happens before在channel c上的发送操作,c的发送操作happens before其相应的接收操作的完成,接收操作的完成happens before打印操作(happened-before本质上是一种偏序关系,所以也是满足传递性的).

**channel的close操作happens before其接收到0值(channel的状态是closed的时候接收操作会返回0值,可以参考runtime/chan.go)**

上面的🌰 中,使用`close(c)`替换`c <- 0`也会得到遵循相同的行为.

**无缓冲channel的接收操作happens before发送操作的完成**

下面是一个用无缓冲channel替换了上述的缓冲channel的🌰
{% highlight golang %}

var c = make(chan int)
var a string

func f() {
	a = "hello, world"
	<-c
}
func main() {
	go f()
	c <- 0
	print(a)
}

{% endhighlight %}

上述代码同样也会打印出"hello, world",a的写操作happens before在channel c上的接收操作,接收操作happens before相应的send操作完成,send操作happends
before打印操作.

如果这个channel是有缓冲的,这个程序不确保能打印出"hello, world".(可能会打印空字符串,崩溃或者其他行为)

**在容量为C的channel中,第k个的接收操作happens before第(k+C)个的发送操作的完成**

这条规则更加统一和概括,使上一条规则同样适用于有缓冲的channel.它允许了在有缓冲的channel上的信号量的计数操作：channel内item的数量对应相应的活跃操作的数量,channel的容量是允许同时活跃操作的最大的数量,发送操作相当于获取信号量,接收操作相当于释放信号量,这是一个限制并发的惯用模式.

This program starts a goroutine for every entry in the work list, but the goroutines coordinate using the limit channel to ensure that at most three are running work functions at a time.

{% highlight golang %}
var limit = make(chan int, 3)

func main() {
	for _, w := range work {
		go func(w func()) {
			limit <- 1
			w()
			<-limit
		}(w)
	}
	select{}
}
{% endhighlight %}
