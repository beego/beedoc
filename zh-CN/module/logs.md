---
name: logs 模块
sort: 3
---

# 日志处理

这是一个用来处理日志的库，它的设计思路来自于 `database/sql`，目前支持的引擎有 file、console、net、smtp，可以通过如下方式进行安装：

	go get github.com/astaxie/beego/logs

## 如何使用

### 通用方式
首先引入包：

	import (
		"github.com/astaxie/beego/logs"
	)	

然后添加输出引擎（log 支持同时输出到多个引擎），这里我们以 console 为例，第一个参数是引擎名（包括：console、file、conn、smtp、es、multifile）

	logs.SetLogger("console")
	
添加输出引擎也支持第二个参数,用来表示配置信息，详细的配置请看下面介绍：
    
    logs.SetLogger(logs.AdapterFile,`{"filename":"project.log","level":7,"maxlines":0,"maxsize":0,"daily":true,"maxdays":10}`)

然后我们就可以在我们的逻辑中开始任意的使用了：


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

### 多个实例
一般推荐使用通用方式进行日志，但依然支持单独声明来使用独立的日志

        package main
        
        import (
        	"github.com/astaxie/beego/logs"
        )
        
        func main() {
        	log := logs.NewLogger()
        	log.SetLogger(logs.AdapterConsole)
        	log.Debug("this is a debug message")
        }


	
## 输出文件名和行号

日志默认不输出调用的文件名和文件行号,如果你期望输出调用的文件名和文件行号,可以如下设置

	logs.EnableFuncCallDepth(true)
	
开启传入参数true,关闭传入参数false,默认是关闭的.

如果你的应用自己封装了调用log包,那么需要设置SetLogFuncCallDepth,默认是2,也就是直接调用的层级,如果你封装了多层,那么需要根据自己的需求进行调整.

	logs.SetLogFuncCallDepth(3)
	
## 异步输出日志

为了提升性能, 可以设置异步输出:

    logs.Async()
    
异步输出允许设置缓冲chan的大小

    logs.Async(1e3)

## 引擎配置设置

- console
   
	可以设置输出的级别，或者不设置保持默认，默认输出到 `os.Stdout`：
		logs.SetLogger("console", `{"level":1}`)						
- file 

	设置的例子如下所示：
	
		logs.SetLogger("file", `{"filename":"test.log"}`)
		
	主要的参数如下说明：
	- filename 保存的文件名
	- maxlines 每个文件保存的最大行数，默认值 1000000
	- maxsize 每个文件保存的最大尺寸，默认值是 1 << 28, //256 MB
	- daily 是否按照每天 logrotate，默认是 true
	- maxdays 文件最多保存多少天，默认保存 7 天
	- rotate 是否开启 logrotate，默认是 true
	- level 日志保存的时候的级别，默认是 Trace 级别
	- perm 日志文件权限

- multifile 

	设置的例子如下所示：
	
		logs.SetLogger("multifile", ``{"filename":"test.log","separate":["emergency", "alert", "critical", "error", "warning", "notice", "info", "debug"]}``)
		
	主要的参数如下说明(除separate外,均与file相同)：
	- filename 保存的文件名
	- maxlines 每个文件保存的最大行数，默认值 1000000
	- maxsize 每个文件保存的最大尺寸，默认值是 1 << 28, //256 MB
	- daily 是否按照每天 logrotate，默认是 true
	- maxdays 文件最多保存多少天，默认保存 7 天
	- rotate 是否开启 logrotate，默认是 true
	- level 日志保存的时候的级别，默认是 Trace 级别
	- perm 日志文件权限
	- separate 需要单独写入文件的日志级别,设置后命名类似test.error.log
	
	
- conn

	网络输出，设置的例子如下所示：
	
		logs.SetLogger("conn", `{"net":"tcp","addr":":7020"}`)
		
	主要的参数说明如下：
	- reconnectOnMsg 是否每次链接都重新打开链接，默认是 false
	- reconnect 是否自动重新链接地址，默认是 false
	- net 发开网络链接的方式，可以使用 tcp、unix、udp 等
	- addr 网络链接的地址
	- level  日志保存的时候的级别，默认是 Trace 级别
	
- smtp

	邮件发送，设置的例子如下所示：
	
		logs.SetLogger("smtp", `{"username":"beegotest@gmail.com","password":"xxxxxxxx","host":"smtp.gmail.com:587","sendTos":["xiemengjun@gmail.com"]}`)	
		
	主要的参数说明如下：
	- username smtp 验证的用户名
	- password smtp 验证密码
	- host  发送的邮箱地址
	- sendTos   邮件需要发送的人，支持多个
	- subject   发送邮件的标题，默认是 `Diagnostic message from server`
	- level 日志发送的级别，默认是 Trace 级别

- ElasticSearch 
    
    输出到 ElasticSearch:
    
   		logs.SetLogger("es", `{"dsn":"http://localhost:9200/","level":1}`)
