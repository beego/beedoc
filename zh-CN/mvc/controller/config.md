---
name: 参数配置
sort: 1
---

# 参数配置

beego 目前支持 INI、XML、JSON、YAML 格式的配置文件解析，但是默认采用了 INI 格式解析，用户可以通过简单的配置就可以获得很大的灵活性。

## 默认配置解析

beego 默认会解析当前应用下的 `conf/app.conf` 文件。

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

### 不同级别的配置

在配置文件里面支持section，可以有不同的Runmode的配置，默认优先读取runmode下的配置信息，例如下面的配置文件：

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090
	runmode ="dev"
	autorender = false
	autorecover = false
	viewspath = "myview"
	
	[dev]
	httpport = 8080
	[prod]
	httpport = 8088
	[test]
	httpport = 8888
	
上面的配置文件就是在不同的runmode下解析不同的配置，例如在dev模式下，httpport是8080，在prod模式下是8088，在test模式下是8888.其他配置文件同理。解析的时候优先解析runmode下地配置，然后解析默认的配置。

### 系统默认参数

beego 中带有很多可配置的参数，我们来一一认识一下它们，这样有利于我们在接下来的 beego 开发中可以充分的发挥他们的作用(你可以通过在`conf/app.conf`中设置对应的值，大小写都可以来重写这些默认值)：

* AppName

	应用名称，默认是 beego。通过`bee new`创建的是创建的项目名。

* AppPath

	当前应用的路径，默认会通过设置`os.Args[0]`获得执行的命令的第一个参数，所以你在使用 supervisor 管理进程的时候记得采用全路径启动。 	
	
* AppConfigPath

	配置文件所在的路径，默认是应用程序对应的目录下的 `conf/app.conf`，用户可以修改该值配置自己的配置文件。

* EnableHttpListen
 
	是否启用HTTP监听，默认是true

* HttpAddr

	应用监听地址，默认为空，监听所有的网卡 IP。

* HttpPort

	应用监听端口，默认为 8080。
	
* EnableHttpTLS

	是否启用 HTTPS，默认是关闭。
	
* HttpsPort

	应用监听https端口，默认为 10443。	
	
* HttpCertFile
	
	开启 HTTPS 之后，certfile 的路径。

* HttpKeyFile		

	开启 HTTPS 之后，keyfile 的路径。

* HttpServerTimeOut

	设置 HTTP 的超时时间，默认是 0，不超时。	
	
* RunMode

	应用的模式，默认是 dev，为开发模式，在开发模式下出错会提示友好的出错页面，如前面错误描述中所述。

* AutoRender

	是否模板自动渲染，默认值为 true，对于 API 类型的应用，应用需要把该选项设置为 false，不需要渲染模板。

* RecoverPanic

	是否异常恢复，默认值为 true，即当应用出现异常的情况，通过 recover 恢复回来，而不会导致应用异常退出。	

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

* SessionHashFunc

	sessionID 生成函数，默认是 sha1。
	
* SessionHashKey

	session hash 的 key。
	
* SessionCookieLifeTime

	session 默认存在客户端的 cookie 的时间，默认值是 3600 秒。			
* UseFcgi

	是否启用 fastcgi，默认是 false。

* MaxMemory

	文件上传默认内存缓存大小，默认值是 `1 << 26`(64M)。

* EnableGzip

	是否开启 gzip 支持，默认为 false 不支持 gzip，一旦开启了 gzip，那么在模板输出的内容会进行 gzip 或者 zlib 压缩，根据用户的 Accept-Encoding 来判断。

* DirectoryIndex

	是否开启静态目录的列表显示，默认不显示目录，返回 403 错误。
	
* BeegoServerName

	beego 服务器默认在请求的时候输出 server 为 beego。
	
* EnableAdmin

	是否开启进程内监控模块，默认关闭。
	
* AdminHttpAddr

	监控程序监听的地址，默认值是 `localhost`。
	
* AdminHttpPort
	
	监控程序监听的端口，默认值是 8088。
	
* TemplateLeft

	模板左标签，默认值是`{{`。
	
* TemplateRight

	模板右标签，默认值是`}}`。
	
* ErrorsShow

	是否显示错误，默认显示错误信息。

* EnableXSRF
  开启 XSRF

* XSRFKEY

	XSRF 的 key 信息，默认值是 beegoxsrf。
	
* XSRFExpire

	XSRF 过期时间，默认值是 0。	
	
* FlashName

	Flash数据设置时Cookie的名称，默认是BEEGO_FLASH

* FlashSeperator

	Flash数据的分隔符，默认是BEEGOFLASH

* StaticDir

	静态文件目录设置，默认是static	
	1. 单个目录，相同于`beego.SetStaticPath("/download","download")`
	
		StaticDir = download
	2. 多个目录，相当于`beego.SetStaticPath("/download","down")`和`beego.SetStaticPath("/download2","down2")`
	
		StaticDir = download:down download2:down2	
		
 * EnableDocs

	是否开启文档内置功能，默认是false		
	
 * AppConfigProvider

	配置文件的格式，默认是ini，可以配置为xml，yaml，json					 			