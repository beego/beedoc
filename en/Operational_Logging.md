# Logging

Beego has a default BeeLogger object that outputs log into stdout, and you can use your own logger as well:

	beego.SetLogger(*log.Logger)

Now Beego supports new way to record your log with automatically log rotate. Use following code in your main function:

	filew := beego.NewFileWriter("tmp/log.log", true)
	err := filew.StartLogger()
	if err != nil {
		beego.Critical("NewFileWriter err", err)
	}

So Beego records your log into file `tmp/log.log`, the second argument indicates whether enable log rotate or not. The rules of rotate as follows:

1. segment log every 1,000,000 lines.
2. segment log every 256 MB file size.
3. segment log daily. 
4. save log file up to 7 days as default.

You cannot segment log over 999 times everyday, the segmented file name with format `<defined file name>.<date>.<three digits>`.

You are able to modify rotate rules with following methods, be sure that you call them before `StartLogger()`.

- func (w *FileLogWriter) SetRotateDaily(daily bool) *FileLogWriter
- func (w *FileLogWriter) SetRotateLines(maxlines int) *FileLogWriter
- func (w *FileLogWriter) SetRotateMaxDays(maxdays int64) *FileLogWriter
- func (w *FileLogWriter) SetRotateSize(maxsize int) *FileLogWriter

### Different levels of log

* Trace(v ...interface{})
* Debug(v ...interface{})
* Info(v ...interface{})
* Warn(v ...interface{})
* Error(v ...interface{})
* Critical(v ...interface{})

You can use following code to set log level:

	beego.SetLevel(beego.LevelError)

Your project may have a lot of log outputs, but you don't want to output everything after your application is running on the internet, for example, you want to ignore Trace, Debug and Info level log outputs, you can use following setting:

	beego.SetLevel(beego.LevelWarning)

Then Beego will not output log that has lower level of LevelWarning. Here is the list of all log levels, order from lower to higher:

LevelTrace, LevelDebug, LevelInfo, LevelWarning, LevelError, LevelCritical

You can use different log level to output different error messages, it's based on how critical the error you think it is:


### Examples of log messages

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


### Example

	func internalCalculationFunc(x, y int) (result int, err error) {
		beego.Debug("calculating z. x:",x," y:",y)
		z := y
		switch {
		case x == 3 :
			beego.Trace("x == 3")
			panic("Failure.")
		case y == 1 :
			beego.Trace("y == 1")
			return 0, errors.New("Error!")
		case y == 2 :
			beego.Trace("y == 2")
			z = x
		default :
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
