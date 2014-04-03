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
	beego.Critcal("this is critcal")

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
	
## 输出文件名和行号

日志默认不输出调用的文件名和文件行号,如果你期望输出调用的文件名和文件行号,可以如下设置

	beego.SetLogFuncCall(true)
	
开启传入参数true,关闭传入参数false,默认是关闭的.			
	
## 完整示例

```go
func internalCalculationFunc(x, y int) (result int, err error) {
    beego.Debug("calculating z. x:", x, " y:", y)
    z := y
    switch {
    case x == 3:
        beego.Trace("x == 3")
        panic("Failure.")
    case y == 1:
        beego.Trace("y == 1")
        return 0, errors.New("Error!")
    case y == 2:
        beego.Trace("y == 2")
        z = x
    default:
        beego.Trace("default")
        z += x
    }
    retVal := z - 3
    beego.Debug("Returning ", retVal)
    
    return retVal, nil
}

func processInput(input inputData) {
    defer func() {
        if r := recover(); r != nil {
            beego.Error("Unexpected error occurred: ", r)
            outputs <- outputData{result: 0, error: true}
        }
    }()
    beego.Info("Received input signal. x:", input.x, " y:", input.y)
    
    res, err := internalCalculationFunc(input.x, input.y)
    if err != nil {
        beego.Warn("Error in calculation:", err.Error())
    }
    
    beego.Info("Returning result: ", res, " error: ", err)
    outputs <- outputData{result: res, error: err != nil}
}

func main() {
    inputs = make(chan inputData)
    outputs = make(chan outputData)
    criticalChan = make(chan int)
    beego.Info("App started.")
    
    go consumeResults(outputs)
    beego.Info("Started receiving results.")
    
    go generateInputs(inputs)
    beego.Info("Started sending signals.")
    
    for {
        select {
        case input := <-inputs:
            processInput(input)
        case <-criticalChan:
            beego.Critical("Caught value from criticalChan: Go shut down.")
            panic("Shut down due to critical fault.")
        }
    }
}
```
