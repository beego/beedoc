---
name: admin 模块
sort: 7
---

这个模块主要是参考了 Dropwizard 框架，是一位用户提醒我说有这么一个框架，然后里面实现一些很酷的东西。那个 [issue](https://github.com/beego/beego/issues/128) 详细描述了该功能的雏形，然后就在参考该功能的情况下增加了一些额外的很酷的功能，接下来我将一一介绍这个模块中的几个功能：健康检查、性能调试、访问统计、计划任务。

在 v2.x 里面，我们将原本的`toolbox`拆分为两块，一块是`admin`，即治理模块；另外一块是`task`。

我们的设计哲学认为，`task`是一个和`server`,`client`平级的东西。

## 如何安装

	go get github.com/beego/beego/v2/core/admin

## healthcheck

监控检查是用于当你应用于产品环境中进程，检查当前的状态是否正常，例如你要检查当前数据库是否可用，如下例子所示：

```go
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
admin.AddHealthCheck("database",&DatabaseCheck{})
```

加入之后，你可以往你的管理端口 `/healthcheck` 发送GET请求：

	$ curl http://beego.me:8088/healthcheck
	* deadlocks: OK
	* database: OK

如果检测显示是正确的，那么输出 OK，如果检测出错，显示出错的信息。

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
		_/Users/astaxie/github/beego/admin.ProcessInput(0x2c89f0, 0x10, 0x634238, 0xc210000008)
			/Users/astaxie/github/beego/admin/profile.go:26 +0x256
		_/Users/astaxie/github/beego/admin.TestProcessInput(0xc21004e090)
			/Users/astaxie/github/beego/admin/profile_test.go:9 +0x5a
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
			_/Users/astaxie/github/beego/admin/_test/_testmain.go:53 +0x9c

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

注意！在 v2.x 里面，我们实际上并不建议直接使用`admin`来收集这种统计信息。只有在你的应用是单机应用，并且只部署少量实例的时候，可以考虑使用这种方式。

我们推荐的是使用专门的中间件来完成统计的功能，例如`opentracing`和`prometheus`。

我们在 web, httplib, orm 上都提供了对应的`Filter`来接入这种中间件。

![](../images/toolbox.jpg)

如何使用这个统计呢？如下所示添加统计：

	admin.StatisticsMap.AddStatistics("POST", "/api/user", "&admin.user", time.Duration(2000))
	admin.StatisticsMap.AddStatistics("POST", "/api/user", "&admin.user", time.Duration(120000))
	admin.StatisticsMap.AddStatistics("GET", "/api/user", "&admin.user", time.Duration(13000))
	admin.StatisticsMap.AddStatistics("POST", "/api/admin", "&admin.user", time.Duration(14000))
	admin.StatisticsMap.AddStatistics("POST", "/api/user/astaxie", "&admin.user", time.Duration(12000))
	admin.StatisticsMap.AddStatistics("POST", "/api/user/xiemengjun", "&admin.user", time.Duration(13000))
	admin.StatisticsMap.AddStatistics("DELETE", "/api/user", "&admin.user", time.Duration(1400))

获取统计信息

	admin.StatisticsMap.GetMap(os.Stdout)

输出如下格式的信息：
```bash
	| requestUrl                                        | method     | times            | used             | max used         | min used         | avg used         |
	| /api/user                                         | POST       |  2               | 122.00us         | 120.00us         | 2.00us           | 61.00us          |
	| /api/user                                         | GET        |  1               | 13.00us          | 13.00us          | 13.00us          | 13.00us          |
	| /api/user                                         | DELETE     |  1               | 1.40us           | 1.40us           | 1.40us           | 1.40us           |
	| /api/admin                                        | POST       |  1               | 14.00us          | 14.00us          | 14.00us          | 14.00us          |
	| /api/user/astaxie                                 | POST       |  1               | 12.00us          | 12.00us          | 12.00us          | 12.00us          |
	| /api/user/xiemengjun                              | POST       |  1               | 13.00us          | 13.00us          | 13.00us          | 13.00us          |
```
