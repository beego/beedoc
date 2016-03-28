---
name: Logs Module
sort: 3
---

# Logging

The logging module is inspired by `database/sql`. It supports file, console, net and smtp as destination providers by default. It is installed like this:

	go get github.com/astaxie/beego/logs

## Basic Usage

### General Usage
Import package:

	import (
		"github.com/astaxie/beego/logs"
	)

Initialize log variable (10000 is the cache size):

	log := logs.NewLogger(10000)

Then add the output provider (it supports outputting to multiple providers at the same time). The first parameter is the provider name (`console`, `file`,`multifile`, `conn` , `smtp` or `es`).

	log.SetLogger("console")
	
The second parameter is a provider-specific configuration string (see below for details).
    logs.SetLogger(logs.AdapterFile,`{"filename":"project.log","level":7,"maxlines":0,"maxsize":0,"daily":true,"maxdays":10}`)


Then we can use it in our code:


    package main
    
    import (
    	"github.com/astaxie/beego/logs"
    )
    
    func main() {    
    	//an official log.Logger
    	l := logs.GetLogger()
    	l.Println("this is a message of http")
    	//an official log.Logger with prefix ORM
    	logs.GetLogger("ORM").Println("this is a message of orm")
    
        logs.Debug("my book is bought in the year of ", 2016)
     	logs.Info("this %s cat is %v years old", "yellow", 3)
     	logs.Warn("json is a type of kv like", map[string]int{"key": 2016})
       	logs.Error(1024, "is a very", "good game")
       	logs.Critical("oh,crash")
    }
 
### Another Way

beego/logs supports to declare a single logger to use
    
        
        package main
        
        import (
        	"github.com/astaxie/beego/logs"
        )
        
        func main() {
        	log := logs.NewLogger()
        	log.SetLogger(logs.AdapterConsole)
        	log.Debug("this is a debug message")
        }

## Logging caller information (file name & line number)

The module can be configured to include the file & line number of the log calls in the logging output. This functionality is disabled by default, but can be enabled using the following code:

	logs.EnableFuncCallDepth(true)

Use `true` to turn file & line number logging on, and `false` to turn it off. Default is `false`.

If your application encapsulates the call to the log methods, you may need use `SetLogFuncCallDepth` to set the number of stack frames to be skipped before the caller information is retrieved. The default is 2.

	logs.SetLogFuncCallDepth(3)
	
## Logging asynchronously

You can set logger to asynchronous logging to improve performance:

    logs.Async()

Add a parameter to set the length of buffer channel
    logs.Async(1e3)

## Provider configuration

Each provider supports a set of configuration options.

- console

	Can set output level or use default. Uses `os.Stdout` by default.

		logs.SetLogger("console", `{"level":1}`)

- file

	E.g.:

		logs.SetLogger("file", `{"filename":"test.log"}`)

	Parameters:
	- filename: Save to filename.
	- maxlines: Maximum lines for each log file, 1000000 by default.
	- maxsize: Maximum size of each log file, 1 << 28 or 256M by default.
	- daily: If log rotates by day, true by default.
	- maxdays: Maximum number of days log files will be kept, 7 by default.
	- rotate: Enable logrotate or not, true by default.
	- level: Log level, Trace by default.
	- perm: Log file permission
	
- multifile 

	E.g.:

		logs.SetLogger("multifile", ``{"filename":"test.log","separate":["emergency", "alert", "critical", "error", "warning", "notice", "info", "debug"]}``)

	Parameters:
	- filename: Save to filename.
	- maxlines: Maximum lines for each log file, 1000000 by default.
	- maxsize: Maximum size of each log file, 1 << 28 or 256M by default.
	- daily: If log rotates by day, true by default.
	- maxdays: Maximum number of days log files will be kept, 7 by default.
	- rotate: Enable logrotate or not, true by default.
	- level: Log level, Trace by default.
	- perm: Log file permission
	- separate: Log file will separate to test.error.log/test.debug.log as the log level set in the json array

- conn

	Net output:

		logs.SetLogger("conn", `{"net":"tcp","addr":":7020"}`)

	Parameters:
	- reconnectOnMsg: If true: reopen and close connection every time a message is sent. False by default.
	- reconnect: If true: auto connect. False by default.
	- net: connection type: tcp, unix or udp.
	- addr: net connection address.
	- level: Log level, Trace by default.

- smtp

	Log by email:

		logs.SetLogger("smtp", `{"username":"beegotest@gmail.com","password":"xxxxxxxx","host":"smtp.gmail.com:587","sendTos":["xiemengjun@gmail.com"]}`)

	Parameters:
	- username: smtp username.
	- password: smtp password.
	- host: SMTP server host.
	- sendTos: email addresses to which the logs will be sent.
	- subject: email subject, `Diagnostic message from server` by default.
	- level: Log level, Trace by default.
	
	
- ElasticSearch 
    
    Log to ElasticSearch:
    
   		logs.SetLogger("es", `{"dsn":"http://localhost:9200/","level":1}`)
