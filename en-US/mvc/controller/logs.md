---
name: Logging
sort: 11
---

# Logging

We have already spoken about how Beego is based on several independent modules. Beego uses the logs module to handle logging. Beego already has a variable `BeeLogger` which is a `logs.BeeLogger` type and initialized as `console` which will output to `console`.

## Basic usage

In our Beego application we log as below:

	beego.Debug("this is debug")
	beego.Info("this is info")
	beego.Notice("this is notice")
	beego.Warn("this is warn")
	beego.Error("this is error")
	beego.Critical("this is critical")
	beego.Alert("this is alert")
	beego.Emergency("this is emergency")

## Configure output

Usually we want to output information into a log file and it's easy to do that:

	beego.SetLogger("file", `{"filename":"logs/test.log"}`)

For more usage of logs, see [logs](../../module/logs.md).
​	
After the above setting has been made, logs will output to both console and file. If you only want to output to file, you need to remove console like this:

	beego.BeeLogger.DelLogger("console")	


## Configure logging level

As we saw above, there are 8 different logging levels:	
	LevelDebug
	LevelInformational
	LevelNotice
	LevelWarning
	LevelError
	LevelCritical
	LevelAlert
	LevelEmergency

The logging level goes from debug to emergency. It will output all by default. We can set the different logging levels on different servers:

	beego.SetLevel(beego.LevelInformational)

## output file and line number

Log output does not include call file by default, if you're expecting output calls' file name and line number, set as follows

	beego.SetLogFuncCall(true)

Default is false, i.e. not to log file names and line numbers.	

## Example

```go
func internalCalculationFunc(x, y int) (result int, err error) {
	beego.Debug("calculating z. x:",x," y:",y)
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
	retVal := z-3
	beego.Debug("Returning ", retVal)
	
	return retVal, nil
}	

func processInput(input inputData) {
	defer func() {
		if r := recover(); r != nil {
			beego.Error("Unexpected error occurred: ", r)
			outputs <- outputData{result : 0, error : true}
		}
	}()
	beego.Info("Received input signal. x:",input.x," y:", input.y)
	
	res, err := internalCalculationFunc(input.x, input.y)
	if err != nil {
		beego.Warn("Error in calculation:", err.Error())
	}
	
	beego.Info("Returning result: ",res," error: ",err)
	outputs <- outputData{result : res, error : err != nil}
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
			case input := <- inputs:
				processInput(input)
			case <- criticalChan:
				beego.Critical("Caught value from criticalChan: Go shut down.")
				panic("Shut down due to critical fault.")
		}	
	}
}
```
