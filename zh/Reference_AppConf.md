# 配置管理

## 自定义配置解析

beego 目前采用了模块化设计，config 独立出来了一个模块，你可以通过如下方式进行安装：

	go get github.com/astaxie/beego/config
	
目前实现了四个文件格式的解析：ini、json、xml、yaml

具体的使用例子如下所示：	


```go
func Post() {
	iniconf, err := config.NewConfig("ini", "testini.conf")
	if err != nil {
		t.Fatal(err)
	}
	if iniconf.String("appname") != "beeapi" {
		t.Fatal("appname not equal to beeapi")
	}
	if port, err := iniconf.Int("httpport"); err != nil || port != 8080 {
		t.Error(port)
		t.Fatal(err)
	}
	if port, err := iniconf.Int64("mysqlport"); err != nil || port != 3600 {
		t.Error(port)
		t.Fatal(err)
	}
	if pi, err := iniconf.Float("PI"); err != nil || pi != 3.1415976 {
		t.Error(pi)
		t.Fatal(err)
	}
	if iniconf.String("runmode") != "dev" {
		t.Fatal("runmode not equal to dev")
	}
	if v, err := iniconf.Bool("autorender"); err != nil || v != false {
		t.Error(v)
		t.Fatal(err)
	}
	if v, err := iniconf.Bool("copyrequestbody"); err != nil || v != true {
		t.Error(v)
		t.Fatal(err)
	}
	if err = iniconf.Set("name", "astaxie"); err != nil {
		t.Fatal(err)
	}
	if iniconf.String("name") != "astaxie" {
		t.Fatal("get name error")
	}
}
```

上面这个例子演示了如何使用 beego 的 config 模块，主要是通过 `config.NewConfig` 初始化一个对象，在业务逻辑中就可以通过如下的接口进行数据的读取：

* Set(key, val string) error
* String(key string) string
* Int(key string) (int, error)
* Int64(key string) (int64, error)
* Bool(key string) (bool, error)
* Float(key string) (float64, error)
* DIY(key string) (interface{}, error)

config模块中最重要的是如下这个函数:

	config.NewConfig(adapterName, fileaname string)
	
- 第一个参数表示解析的文件格式，目前支持四种：ini、json、xml、yaml
- 第二个参数表示文件名

## 默认配置解析

beego 支持解析 ini 文件, beego 默认会解析当前应用下的 `conf/app.conf` 文件。

通过这个文件你可以初始化很多 beego 的默认参数：

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090
	runmode ="dev"
	autorender = false
	autorecover = false
	viewspath = "myview"

上面这些参数会替换 beego 默认的一些参数。

你可以在配置文件中配置应用需要用的一些配置信息，例如下面所示的数据库信息：

	mysqluser = "root"
	mysqlpass = "rootpass"
	mysqlurls = "127.0.0.1"
	mysqldb   = "beego"

那么你就可以通过如下的方式获取设置的配置信息:

	beego.AppConfig.String("mysqluser")
	beego.AppConfig.String("mysqlpass")
	beego.AppConfig.String("mysqlurls")
	beego.AppConfig.String("mysqldb")

AppConfig 支持如下方法

- Bool(key string) (bool, error)
- Int(key string) (int, error)
- Int64(key string) (int64, error)
- Float(key string) (float64, error)
- String(key string) string

### 系统默认参数

Beego 中带有很多可配置的参数，我们来一一认识一下它们，这样有利于我们在接下来的 Beego 开发中可以充分的发挥他们的作用：

* BeeApp

	Beego 默认启动的一个应用器入口，在应用 import beego 的时候，在 init 中已经初始化的。

* AppConfig

	Beego 的配置文件解析之后的对象，也是在 init 的时候初始化的，里面保存有解析 `conf/app.conf` 下面所有的参数数据。

* AppConfigPath

	配置文件所在的路径，默认是应用程序对应的目录下的 `conf/app.conf`，用户可以修改该值配置自己的配置文件。

* HttpAddr

	应用监听地址，默认为空，监听所有的网卡 IP。

* HttpPort

	应用监听端口，默认为 8080。

* AppName

	应用名称，默认是 Beego。

* RunMode

	应用的模式，默认是 dev，为开发模式，在开发模式下出错会提示友好的出错页面，如前面错误描述中所述。

* AutoRender

	是否模板自动渲染，默认值为 true，对于 API 类型的应用，应用需要把该选项设置为 false，不需要渲染模板。

* RecoverPanic

	是否异常恢复，默认值为 true，即当应用出现异常的情况，通过 recover 恢复回来，而不会导致应用异常退出。

* PprofOn

	是否启用 pprof，默认是 false，当开启之后，用户可以通过如下地址查看相应的 goroutine 执行情况：
	
		/debug/pprof
		/debug/pprof/cmdline
		/debug/pprof/profile
		/debug/pprof/symbol 

	关于pprof的信息，请参考官方的描述 [pprof](http://gowalker.org/net/http/pprof)。	

* ViewsPath

	模板路径，默认值是 views。

* SessionOn

	session 是否开启，默认是 false。

* SessionProvider

	session 的引擎，默认是 memory。

* SessionName

	存在客户端的 cookie 名称，默认值是 beegosessionID。

* SessionGCMaxLifetime

	session 过期时间，默认值是 3600 秒。

* SessionSavePath

	session 保存路径，默认是空。

* UseFcgi

	是否启用 fastcgi，默认是 false。

* MaxMemory

	文件上传默认内存缓存大小，默认值是 `1 << 26`(64M)。

* EnableGzip

	是否开启 gzip 支持，默认为 false 不支持 gzip，一旦开启了 gzip，那么在模板输出的内容会进行 gzip 或者 zlib 压缩，根据用户的 Accept-Encoding 来判断。

* DirectoryIndex

	是否开启静态目录的列表显示，默认不显示目录，返回 403 错误。
