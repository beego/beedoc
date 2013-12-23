---
name: Logging
sort: 11
---

# Logging

We've already talked about that Beego is based on several independent modules. Beego uses logs module to handle logging. Beego already has a variable `BeeLogger` which is a `logs.BeeLogger` type and initialized as `console` which will output to `console`.

## Basic usage

In our beego application we output as below:

	beego.Trace("this is trace")
	beego.Debug("this is debug")
	beego.Info("this is info")
	beego.Warn("this is warn")
	beego.Error("this is error")
	beego.Crital("this is crital")

## Configure output

Usually we want to output information into log file and It's easy to do that:

	beego.SetLogger("file", `{"filename":"logs/test.log"}`)

For more useage of logs please go to [logs](../../module/logs.md).
	
After the above setting, logs will output to both console and file. If you only want to output to file, you need to remove console like this:

	beego.BeeLogger.DelLogger("console")	


## Configure logging level

As we saw above, there are 6 different logging level:

	LevelTrace
	LevelDebug
	LevelInfo
	LevelWarn
	LevelError
	LevelCritical

The logging level goes from trival to critical. It will output all by default. We can set the different logging level on different server:

	beego.SetLevel(beego.LevelInfo)
