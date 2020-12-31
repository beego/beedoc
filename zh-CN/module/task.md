---
name: task 模块
sort: 6
---

## task

玩过 linux 的用户都知道有一个计划任务的工具 crontab，我们经常利用该工具来定时的做一些任务，但是有些时候我们的进程内也希望定时的来处理一些事情，例如定时的汇报当前进程的内存信息，goroutine 信息等。或者定时的进行手工触发 GC，或者定时的清理一些日志数据等，所以实现了秒级别的定时任务，首先让我们看看如何使用：

1. 初始化一个任务

		tk1 := task.NewTask("tk1", "0 12 * * * *", func(ctx context.Context) error { fmt.Println("tk1"); return nil })

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

		task.AddTask("tk1", tk1)

4. 开始执行全局的任务

		task.StartTask()
		defer task.StopTask()

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

我们经常需要打印一些参数进行调试，但是默认的参数打印总是不是很完美，也没办法定位代码行之类的，所以 beego 的 task 模块进行了 debug 模块的开发，主要包括了两个函数：

- Display()  直接打印结果到 console
- GetDisplayString() 返回打印的字符串

两个函数的功能一模一样，第一个是直接打印到 console，第二个是返回字符串，方便用户存储到日志或者其他存储。

使用很方便，是 key/value 串出现的，如下示例：

	Display("v1", 1, "v2", 2, "v3", 3)

打印结果如下：

	2013/12/16 23:48:41 [Debug] at TestPrint() [/Users/astaxie/github/beego/task/debug_test.go:13]

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

	2013/12/16 23:48:41 [Debug] at TestPrintPoint() [/Users/astaxie/github/beego/task/debug_test.go:26]

	[Variables]
	v1 = &task.mytype{
	    next: &task.mytype{
	        next: nil,
	        prev: 0x210335420,
	    },
	    prev: nil,
	}
	v2 = &task.mytype{
	    next: nil,
	    prev: &task.mytype{
	        next: 0x210335430,
	        prev: nil,
	    },
	}
