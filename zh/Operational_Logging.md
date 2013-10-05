# 日志记录

beego 默认有一个初始化的 BeeLogger 对象输出内容到 stdout 中，你可以通过如下的方式设置自己的输出，目前 beego 采用了模块化设计，beego 会默认调用 github.com/astaxie/beego/logs 模块，你可以通过如下函数设置输出：

	beego.BeeLogger.SetLogger(adaptername string, config string)

目前支持的 adaptername 支持四种方式的输出：console(beego默认输出)、file、conn、smtp，更多相信的请参考https://github.com/astaxie/beego/tree/master/logs

现在 beego 支持文件方式输出到，而且支持文件的自动化 logrotate，在 main 函数入口处初始化如下：

	beego.BeeLogger.SetLogger("file", `{"filename":"logs/logs.log"}`)

这样就默认开始在当前目录的 logs/logs.log 文件中开始记录日志，默认支持文件的 logrotate，开启的 rotate 的规则如下：

1. 1000000 行日志就自动分割
2. 文件的大小为 256M 就自动分割
3. 每天进行分割
4. 日志默认保存 7 天

一天之中分割不能多余 999 个，每个分割的文件名是 `定义的文件名.日期.三位数字`。

### 不同级别的 log 日志函数

* Trace(v ...interface{})
* Debug(v ...interface{})
* Info(v ...interface{})
* Warn(v ...interface{})
* Error(v ...interface{})
* Critical(v ...interface{})

你可以通过下面的方式设置不同的日志分级：

	beego.SetLevel(beego.LevelError)

当你代码中有很多日志输出之后，如果想上线，但是你不想输出 Trace、Debug、Info 等信息，那么你可以设置如下：

	beego.SetLevel(beego.LevelWarning)

这样的话就不会输出小于这个 level 的日志，日志的排序如下：

LevelTrace、LevelDebug、LevelInfo、LevelWarning、LevelError、LevelCritical

用户可以根据不同的级别输出不同的错误信息，如下例子所示：

### 日志分级记录示例

- Trace

	* "Entered parse function validation block"
	* "Validation: entered second 'if'"
	* "Dictionary 'Dict' is empty. Using default value"

- Debug

	* "Web page requested: http://somesite.com Params='...'"
	* "Response generated. Response size: 10000. Sending."
	* "New file received. Type:PNG Size:20000"

- Info

	* "Web server restarted"
	* "Hourly statistics: Requested pages: 12345 Errors: 123 ..."
	* "Service paused. Waiting for 'resume' call"

- Warn

	* "Cache corrupted for file='test.file'. Reading from back-end"
	* "Database 192.168.0.7/DB not responding. Using backup 192.168.0.8/DB"
	* "No response from statistics server. Statistics not sent"

- Error

	* "Internal error. Cannot process request #12345 Error:...."
	* "Cannot perform login: credentials DB not responding"

- Critical

	* "Critical panic received: .... Shutting down"
	* "Fatal error: ... App is shutting down to prevent data corruption or loss"


### 完整示例

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