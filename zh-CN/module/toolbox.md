---
name: toolbox 模块
sort: 6
---

# 核心工具模块

这个模块主要是参考了 Dropwizard 框架，是一位用户提醒我说有这么一个框架，然后里面实现一些很酷的东西。那个 [issue](https://github.com/astaxie/beego/issues/128) 详细描述了该功能的雏形，然后就在参考该功能的情况下增加了一些额外的很酷的功能，接下来我讲一一，介绍这个模块中的几个功能：健康检查、性能调试、访问统计、计划任务。

## 如何安装

	go get github.com/astaxie/beego/toolbox

## healthcheck

监控检查是用于当你应用于产品环境中进程，检查当前的状态是否正常，例如你要检查当前数据库是否可用，如下例子所示：

```
type DatabaseCheck struct {
}

func (dc *DatabaseCheck) Check() error {
	if dc.isConnected() {
		return nil
	} else {
		return errors.New("can't connect database")
	}
}
```
然后就可以通过如下方式增加检测项：

```
toolbox.AddHealthCheck("database",&DatabaseCheck{})
```

加入之后，你可以往你的管理端口`/healthcheck`发送GET请求：

	$ curl http://beego.me:8088/healthcheck
	* deadlocks: OK
	* database: OK
	
如果检测显示是正确的，那么输出OK，如果检测出错，显示出错的信息。
	
## profile

对于运行中的进程的性能监控是我们进行程序调优和查找问题的最佳方法，例如 GC、goroutine 等基础信息。profile 提供了方便的入口方便用户来调试程序，他主要是通过入口函数 `ProcessInput` 来进行处理各类请求，主要包括以下几种调试：

- lookup goroutine

	打印出来当前全部的 goroutine 执行的情况，非常方便查找各个 goroutine 在做的事情：
	
		goroutine 3 [running]:
		runtime/pprof.writeGoroutineStacks(0x634238, 0xc210000008, 0x62b000, 0xd200000000000000)
			/Users/astaxie/go/src/pkg/runtime/pprof/pprof.go:511 +0x7c
		runtime/pprof.writeGoroutine(0x634238, 0xc210000008, 0x2, 0xd2676410957b30fd, 0xae98)
			/Users/astaxie/go/src/pkg/runtime/pprof/pprof.go:500 +0x3c
		runtime/pprof.(*Profile).WriteTo(0x52ebe0, 0x634238, 0xc210000008, 0x2, 0x1, ...)
			/Users/astaxie/go/src/pkg/runtime/pprof/pprof.go:229 +0xb4
		_/Users/astaxie/github/beego/toolbox.ProcessInput(0x2c89f0, 0x10, 0x634238, 0xc210000008)
			/Users/astaxie/github/beego/toolbox/profile.go:26 +0x256
		_/Users/astaxie/github/beego/toolbox.TestProcessInput(0xc21004e090)
			/Users/astaxie/github/beego/toolbox/profile_test.go:9 +0x5a
		testing.tRunner(0xc21004e090, 0x532320)
			/Users/astaxie/go/src/pkg/testing/testing.go:391 +0x8b
		created by testing.RunTests
			/Users/astaxie/go/src/pkg/testing/testing.go:471 +0x8b2
		
		goroutine 1 [chan receive]:
		testing.RunTests(0x315668, 0x532320, 0x4, 0x4, 0x1)
			/Users/astaxie/go/src/pkg/testing/testing.go:472 +0x8d5
		testing.Main(0x315668, 0x532320, 0x4, 0x4, 0x537700, ...)
			/Users/astaxie/go/src/pkg/testing/testing.go:403 +0x84
		main.main()
			_/Users/astaxie/github/beego/toolbox/_test/_testmain.go:53 +0x9c

- lookup heap

	用来打印当前 heap 的信息：
	
		heap profile: 1: 288 [2: 296] @ heap/1048576
		1: 288 [2: 296] @
		
		
		# runtime.MemStats
		# Alloc = 275504
		# TotalAlloc = 275512
		# Sys = 4069608
		# Lookups = 5
		# Mallocs = 469
		# Frees = 1
		# HeapAlloc = 275504
		# HeapSys = 1048576
		# HeapIdle = 647168
		# HeapInuse = 401408
		# HeapReleased = 0
		# HeapObjects = 468
		# Stack = 24576 / 131072
		# MSpan = 4472 / 16384
		# MCache = 1504 / 16384
		# BuckHashSys = 1476472
		# NextGC = 342976
		# PauseNs = [370712 77378 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
		# NumGC = 2
		# EnableGC = true
		# DebugGC = false

- lookup threadcreate

	查看创建线程的信息：
	
		threadcreate profile: total 4
		1 @ 0x17f68 0x183c7 0x186a8 0x188cc 0x19ca9 0xcf41 0x139a3 0x196c0
		#	0x183c7	newm+0x27			/Users/astaxie/go/src/pkg/runtime/proc.c:896
		#	0x186a8	startm+0xb8			/Users/astaxie/go/src/pkg/runtime/proc.c:974
		#	0x188cc	handoffp+0x1ac			/Users/astaxie/go/src/pkg/runtime/proc.c:992
		#	0x19ca9	runtime.entersyscallblock+0x129	/Users/astaxie/go/src/pkg/runtime/proc.c:1514
		#	0xcf41	runtime.notetsleepg+0x71	/Users/astaxie/go/src/pkg/runtime/lock_sema.c:253
		#	0x139a3	runtime.MHeap_Scavenger+0xa3	/Users/astaxie/go/src/pkg/runtime/mheap.c:463
		
		1 @ 0x17f68 0x183c7 0x186a8 0x188cc 0x189c3 0x1969b 0x2618b
		#	0x183c7	newm+0x27		/Users/astaxie/go/src/pkg/runtime/proc.c:896
		#	0x186a8	startm+0xb8		/Users/astaxie/go/src/pkg/runtime/proc.c:974
		#	0x188cc	handoffp+0x1ac		/Users/astaxie/go/src/pkg/runtime/proc.c:992
		#	0x189c3	stoplockedm+0x83	/Users/astaxie/go/src/pkg/runtime/proc.c:1049
		#	0x1969b	runtime.gosched0+0x8b	/Users/astaxie/go/src/pkg/runtime/proc.c:1382
		#	0x2618b	runtime.mcall+0x4b	/Users/astaxie/go/src/pkg/runtime/asm_amd64.s:178
		
		1 @ 0x17f68 0x183c7 0x170bc 0x196c0
		#	0x183c7	newm+0x27		/Users/astaxie/go/src/pkg/runtime/proc.c:896
		#	0x170bc	runtime.main+0x3c	/Users/astaxie/go/src/pkg/runtime/proc.c:191
		
		1 @

- lookup block

	查看 block 信息：

		--- contention:
		cycles/second=2294781025

- start cpuprof

	开始记录 cpuprof 信息，生产一个文件 cpu-pid.pprof，开始记录当前进程的 CPU 处理信息

- stop cpuprof

	关闭记录信息

- get memprof

	开启记录 memprof，生产一个文件 mem-pid.memprof

- gc summary

	查看 GC 信息
	
		NumGC:2 Pause:54.54us Pause(Avg):170.82us Overhead:177.49% Alloc:248.97K Sys:3.88M Alloc(Rate):1.23G/s Histogram:287.09us 287.09us 287.09us 

## statistics

请先看下面这张效果图，你有什么想法，很酷？是的，很酷，现在 tootlbox 就是支持这样的功能了：

![](../images/toolbox.jpg)

如何使用这个统计呢？如下所示添加统计：

	toolbox.StatisticsMap.AddStatistics("POST", "/api/user", "&admin.user", time.Duration(2000))
	toolbox.StatisticsMap.AddStatistics("POST", "/api/user", "&admin.user", time.Duration(120000))
	toolbox.StatisticsMap.AddStatistics("GET", "/api/user", "&admin.user", time.Duration(13000))
	toolbox.StatisticsMap.AddStatistics("POST", "/api/admin", "&admin.user", time.Duration(14000))
	toolbox.StatisticsMap.AddStatistics("POST", "/api/user/astaxie", "&admin.user", time.Duration(12000))
	toolbox.StatisticsMap.AddStatistics("POST", "/api/user/xiemengjun", "&admin.user", time.Duration(13000))
	toolbox.StatisticsMap.AddStatistics("DELETE", "/api/user", "&admin.user", time.Duration(1400))

获取统计信息	
	
	toolbox.StatisticsMap.GetMap(os.Stdout)	

输出如下格式的信息：

	| requestUrl                                        | method     | times            | used             | max used         | min used         | avg used         |
	| /api/user                                         | POST       |  2               | 122.00us         | 120.00us         | 2.00us           | 61.00us          |
	| /api/user                                         | GET        |  1               | 13.00us          | 13.00us          | 13.00us          | 13.00us          |
	| /api/user                                         | DELETE     |  1               | 1.40us           | 1.40us           | 1.40us           | 1.40us           |
	| /api/admin                                        | POST       |  1               | 14.00us          | 14.00us          | 14.00us          | 14.00us          |
	| /api/user/astaxie                                 | POST       |  1               | 12.00us          | 12.00us          | 12.00us          | 12.00us          |
	| /api/user/xiemengjun                              | POST       |  1               | 13.00us          | 13.00us          | 13.00us          | 13.00us          |	

## task

玩过 linux 的用户都知道有一个计划任务的工具 crontab，我们经常利用该工具来定时的做一些任务，但是有些时候我们的进程内也希望定时的来处理一些事情，例如定时的汇报当前进程的内存信息，goroutine 信息等。或者定时的进行手工触发 GC，或者定时的清理一些日志数据等，所以实现了秒级别的定时任务，首先让我们看看如何使用：

1. 初始化一个任务

		tk1 := toolbox.NewTask("tk1", "0 12 * * * *", func() error { fmt.Println("tk1"); return nil })
	
	函数原型：
	
	NewTask(tname string, spec string, f TaskFunc) *Task	
	- tname 任务名称
	- spec 定时任务格式，请参考下面的详细介绍
	- f 执行的函数 func() error 
	
2. 可以测试开启运行

	可以通过如下的代码运行 TaskFunc，和 spec 无关，用于检测写的函数是否如预期所希望的这样：

		err := tk.Run()
		if err != nil {
			t.Fatal(err)
		}
	
3. 加入全局的计划任务列表	
	
		toolbox.AddTask("tk1", tk1)

4. 开始执行全局的任务

		toolbox.StartTask()
		defer toolbox.StopTask()
		
### spec 详解		

spec 格式是参照 crontab 做的，详细的解释如下所示：


```
//前6个字段分别表示：
//       秒钟：0-59
//       分钟：0-59
//       小时：1-23
//       日期：1-31
//       月份：1-12
//       星期：0-6（0 表示周日）

//还可以用一些特殊符号：
//       *： 表示任何时刻
//       ,：　表示分割，如第三段里：2,4，表示 2 点和 4 点执行
//　　    －：表示一个段，如第三端里： 1-5，就表示 1 到 5 点
//       /n : 表示每个n的单位执行一次，如第三段里，*/1, 就表示每隔 1 个小时执行一次命令。也可以写成1-23/1.
/////////////////////////////////////////////////////////
//	0/30 * * * * *                        每 30 秒 执行
//	0 43 21 * * *                         21:43 执行
//	0 15 05 * * * 　　                     05:15 执行
//	0 0 17 * * *                          17:00 执行
//	0 0 17 * * 1                          每周一的 17:00 执行
//	0 0,10 17 * * 0,2,3                   每周日,周二,周三的 17:00和 17:10 执行
//	0 0-10 17 1 * *                       毎月1日从 17:00 到 7:10 毎隔 1 分钟 执行
//	0 0 0 1,15 * 1                        毎月1日和 15 日和 一日的 0:00 执行
//	0 42 4 1 * * 　 　                     毎月1日的 4:42 分 执行
//	0 0 21 * * 1-6　　                     周一到周六 21:00 执行
//	0 0,10,20,30,40,50 * * * *　           每隔 10 分 执行
//	0 */10 * * * * 　　　　　　              每隔 10 分 执行
//	0 * 1 * * *　　　　　　　　               从 1:0 到 1:59 每隔 1 分钟 执行
//	0 0 1 * * *　　　　　　　　               1:00 执行
//	0 0 */1 * * *　　　　　　　               毎时 0 分 每隔 1 小时 执行
//	0 0 * * * *　　　　　　　　               毎时 0 分 每隔 1 小时 执行
//	0 2 8-20/3 * * *　　　　　　             8:02,11:02,14:02,17:02,20:02 执行
//	0 30 5 1,15 * *　　　　　　              1 日 和 15 日的 5:30 执行
```

## 调试模块(已移动到utils模块)

我们经常需要打印一些参数进行调试，但是默认的参数打印总是不是很完美，也没办法定位代码行之类的，所以 beego的 toolbox 模块进行了 debug 模块的开发，主要包括了两个函数：

- Display()  直接打印结果到 console
- GetDisplayString() 返回打印的字符串

两个函数的功能一模一样，第一个是直接打印到 console，第二个是返回字符串，方便用户存储到日志或者其他存储。

使用很方便，是 key/value 串出现的，如下示例：

	Display("v1", 1, "v2", 2, "v3", 3)
	
打印结果如下：

	2013/12/16 23:48:41 [Debug] at TestPrint() [/Users/astaxie/github/beego/toolbox/debug_test.go:13]
	
	[Variables]
	v1 = 1
	v2 = 2
	v3 = 3	
	
指针类型的打印如下：

	type mytype struct {
		next *mytype
		prev *mytype
	}	
	
	var v1 = new(mytype)
	var v2 = new(mytype)

	v1.prev = nil
	v1.next = v2

	v2.prev = v1
	v2.next = nil

	Display("v1", v1, "v2", v2)

打印结果如下：

	2013/12/16 23:48:41 [Debug] at TestPrintPoint() [/Users/astaxie/github/beego/toolbox/debug_test.go:26]

	[Variables]
	v1 = &toolbox.mytype{
	    next: &toolbox.mytype{
	        next: nil,
	        prev: 0x210335420,
	    },
	    prev: nil,
	}
	v2 = &toolbox.mytype{
	    next: nil,
	    prev: &toolbox.mytype{
	        next: 0x210335430,
	        prev: nil,
	    },
	}		
