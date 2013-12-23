---
name: 日志处理
sort: 11
---

# 日志处理

beego 之前介绍的时候说过是基于几个模块搭建的，beego 的日志处理是基于 logs 模块搭建的，内置了一个变量 `BeeLogger`，默认已经是 `logs.BeeLogger` 类型，初始化了 console，也就是默认输出到 `console`。

## 使用入门

一般在程序中我们使用如下的方式进行输出：

	beego.Trace("this is trace")
	beego.Debug("this is debug")
	beego.Info("this is info")
	beego.Warn("this is warn")
	beego.Error("this is error")
	beego.Crital("this is crital")

## 设置输出

我们的程序往往期望把信息输出到 log 中，现在设置输出到文件很方便，如下所示：

	beego.SetLogger("file", `{"filename":"logs/test.log"}`)

更多详细的日志配置请查看 [日志配置](../../module/logs.md)	
	
这个默认情况就会同时输出到两个地方，一个 console，一个 file，如果只想输出到文件，就需要调用删除操作：

	beego.BeeLogger.DelLogger("console")	

## 设置级别

日志的级别如上所示的代码这样分为六个级别：

	LevelTrace
	LevelDebug
	LevelInfo
	LevelWarn
	LevelError
	LevelCritical

级别依次上升，默认全部打印，但是一般我们在部署环境，可以通过设置级别设置日志级别：

	beego.SetLevel(beego.LevelInfo)