# App.Conf

Beego supports to parse .ini file in path `conf/app.conf`, and you have following options:

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090
	runmode ="dev"
	autorender = false
	autorecover = false
	viewspath = "myview"

If you set value in configuration file, Beego uses it to replace default value.

You can also have other values for your application, for example, database connection information:

	mysqluser = "root"
	mysqlpass = "rootpass"
	mysqlurls = "127.0.0.1"
	mysqldb   = "beego"

Then use following code to load your settings:

	beego.AppConfig.String("mysqluser")
	beego.AppConfig.String("mysqlpass")
	beego.AppConfig.String("mysqlurls")
	beego.AppConfig.String("mysqldb")

AppConfig supports following methods:

- Bool(key string) (bool, error)
- Int(key string) (int, error)
- Int64(key string) (int64, error)
- Float(key string) (float64, error)
- String(key string) string

### Beego arguments

Beego has many configurable arguments, let me introduce to you all of them, so you can use them for more usage in your application:

* BeeApp

	Entry point of Beego, it initialized in init() function when you import Beego package.

* AppConfig

	It stores values from file `conf/app.conf` and initialized in init() function.

* HttpAddr

	Application listening address, default is empty for listening all IP.

* HttpPort

	Application listening port, default is 8080.

* AppName

	Application name, default is "beego".

* RunMode

	Application mode, default is "dev" develop mode and gives friendly error messages.

* AutoRender

	This value indicates whether auto-render or not, default is true, you should set to false for API usage applications.
	
* RecoverPanic

	This value indicates whether recover from panic or not, default is true, and program will not exit when error occurs.
	
* PprofOn

	This value indicates whether enable pprof or not, default is false, and you can use following address to see goroutine execution status once you enable this feature.
	
		/debug/pprof
		/debug/pprof/cmdline
		/debug/pprof/profile
		/debug/pprof/symbol 

	For more information about pprof, please read [pprof](http://golang.org/pkg/net/http/pprof/)

* ViewsPath

	Template path, default is "views".

* SessionOn

	This value indicate whether enable session or not, default is false.

* SessionProvider

	Session engine, default is memory.

* SessionName

	Name for cookie that save in client browser, default is "beegosessionID".

* SessionGCMaxLifetime

	Session expired time, default is 3600 seconds.

* SessionSavePath

	Save path of session, default is empty.

* UseFcgi

	This value indicates whether enable fastcgi or not, default is false.

* MaxMemory

	Maximum memory size for file upload, default is `1 << 26`(64M).

* EnableGzip

	This value indicates whether enable gzip or not, default is false.

* DirectoryIndex

	This value indicates whether to show directory broswer, default is false and return error 403.