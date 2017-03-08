# Bee Watch

[![Build Status](https://drone.io/github.com/beego/beewatch/status.png)](https://drone.io/github.com/beego/beewatch/latest)

Bee Watch 是一款 Go 语言的交互式调试工具。

### 特性

- 使用 `Critical`、`Info` 和 `Trace` 三个级别来改变调试器的行为。
- `Display()` 打印变量值或者使用 `Printf()` 自定义输出格式。
- 使用用户友好界面的 **WebSocket** 模式或能够轻松控制的 **命令行** 模式。
- 调用 `AddWatchVars()` 来监视变量并在程序调用 `Break()` 时显示它们的信息。
- 通过 `beewatch.json` 配置文件来实现自定义配置。

### 安装

Bee Watch 是一个可以使用 “go get” 安装的 Go 项目，您可以执行以下命令完成自动安装：

	go get github.com/beego/beewatch

**注意** 该项目目前只能通过源码安装。

### 快速入门

#### 用法

	package main

	import (
		"time"

		"github.com/beego/beewatch"
	)

	func main() {
		// 默认级别为：Trace，
		// 或使用 `beewatch.Start(beewatch.Info())` 设置级别。
		beewatch.Start()

		// 声明一些变量。
		appName := "Bee Watch"
		boolean := true
		number := 3862
		floatNum := 3.1415926
		test := "fail to watch"

		// 增加监视变量，参数必须为偶数。
		// 需要注意的是您必须传递被监视变量的地址。
		// 该功能当前只支持基本类型。
		// 在下面的例子中，'test' 不会被监视。
		beewatch.AddWatchVars("test", test, "App Name", &appName,
			"bool", &boolean, "number", &number, "float", &floatNum)

		// 打印信息。
		beewatch.Info().Display("App Name", appName).Break().
			Printf("boolean value is %v; number is %d", boolean, number)

		beewatch.Critical().Break().Display("float", floatNum)

		// 修改一些值。
		appName = "Bee Watch2"
		number = 250
		// 您将在这里看到一些有趣的东西。
		beewatch.Trace().Break()

		// 多线程测试。
		for i := 0; i < 3; i++ {
			go multipletest(i)
		}

		beewatch.Trace().Printf("Wait 3 seconds...")
		select {
		case <-time.After(3 * time.Second):
			beewatch.Trace().Printf("Done debug")
		}

		// 关闭调试器，当您调试 Web 服务器时，可忽略该操作。
		beewatch.Close()
	}

	func multipletest(num int) {
		beewatch.Critical().Break().Display("num", num)
	}

#### 连接

当您使用 **WebSocket** 模式时，Bee Watch 会自动监听 [http://localhost:23456](http://localhost:23456)，您可以用过配置 `beewatch.json` 来修改调试器行为（请从 Bee Watch 的源码目录拷贝默认配置）。

您的浏览器必须支持 WebSocket，在 Mac 和 Windows 上的 Chrome、Safari 和 Firefox 均以通过测试。

![Bee Watch demo](https://github.com/beego/beewatch/blob/master/tests/images/demo_beewatch.png?raw=true)

### 示例和 API 文档

[Go Walker](http://gowalker.org/github.com/beego/beewatch)

