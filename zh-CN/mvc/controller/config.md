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
	recoverpanic = false
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

AppConfig 的方法如下：

- Set(key, val string) error
- String(key string) string
- Strings(key string) []string
- Int(key string) (int, error)
- Int64(key string) (int64, error)
- Bool(key string) (bool, error)
- Float(key string) (float64, error)
- DefaultString(key string, defaultVal string) string
- DefaultStrings(key string, defaultVal []string)
- DefaultInt(key string, defaultVal int) int
- DefaultInt64(key string, defaultVal int64) int64
- DefaultBool(key string, defaultVal bool) bool
- DefaultFloat(key string, defaultVal float64) float64
- DIY(key string) (interface{}, error)
- GetSection(section string) (map[string]string, error)
- SaveConfigFile(filename string) error

在使用 ini 类型的配置文件中, key 支持 section::key 模式. 

你可以用 Default* 方法返回默认值.


### 不同级别的配置

在配置文件里面支持section，可以有不同的Runmode的配置，默认优先读取runmode下的配置信息，例如下面的配置文件：

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090
	runmode ="dev"
	autorender = false
	recoverpanic = false
	viewspath = "myview"

	[dev]
	httpport = 8080
	[prod]
	httpport = 8088
	[test]
	httpport = 8888

上面的配置文件就是在不同的runmode下解析不同的配置，例如在dev模式下，httpport是8080，在prod模式下是8088，在test模式下是8888.其他配置文件同理。解析的时候优先解析runmode下的配置，然后解析默认的配置。

读取不同模式下配置参数的方法是“模式::配置参数名”，比如：beego.AppConfig.String("dev::mysqluser")。

对于自定义的参数，需使用beego.GetConfig(typ, key string)来获取指定runmode下的配置（需1.4.0以上版本），typ为参数类型，key为参数名。

### 多个配置文件
INI格式配置支持`include`方式，引用多个配置文件，例如下面的两个配置文件效果同上：

app.conf

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090

	include "app2.conf"

app2.conf

	runmode ="dev"
	autorender = false
	recoverpanic = false
	viewspath = "myview"

	[dev]
	httpport = 8080
	[prod]
	httpport = 8088
	[test]
	httpport = 8888

### 系统默认参数

beego 中带有很多可配置的参数，我们来一一认识一下它们，这样有利于我们在接下来的 beego 开发中可以充分的发挥他们的作用(你可以通过在`conf/app.conf`中设置对应的值，不区分大小写)：

#### 基础配置

* AppConfigPath
配置文件路径，默认是应用程序对应的目录下的 `conf/app.conf`，用户可以在程序代码中修改该值配置自己的配置文件
`beego.AppConfigPath = "conf/app2.conf"`

* AppConfigProvider

	配置文件的格式，默认是ini，可以配置为xml，yaml，json

	`beego.AppConfigProvider = "ini"`

#### App配置

* AppName

	应用名称，默认是 beego。通过`bee new`创建的是创建的项目名。

	`beego.BConfig.AppName = "beego"`

* RunMode

	应用的运行模式，可选值为 `prod`, `dev` 或者 `test`. 默认是 `dev`, 为开发模式，在开发模式下出错会提示友好的出错页面，如前面错误描述中所述。

	`beego.BConfig.RunMode = "dev"`

* RouterCaseSensitive

	是否路由忽略大小写匹配，默认是 true，区分大小写

	`beego.BConfig.RouterCaseSensitive = true`

* ServerName

	beego 服务器默认在请求的时候输出 server 为 beego。

	`beego.BConfig.ServerName = "beego"`

* RecoverPanic  

	是否异常恢复，默认值为 true，即当应用出现异常的情况，通过 recover 恢复回来，而不会导致应用异常退出。

	`beego.BConfig.RecoverPanic = true`

* CopyRequestBody

	是否允许在HTTP请求时，返回原始请求体数据字节，默认为 true （GET or HEAD or 上传文件请求除外）。

	`beego.BConfig.CopyRequestBody = true`


* EnableGzip

	是否开启 gzip 支持，默认为 false 不支持 gzip，一旦开启了 gzip，那么在模板输出的内容会进行 gzip 或者 zlib 压缩，根据用户的 Accept-Encoding 来判断。

	`beego.BConfig.EnableGzip = false`
	
	Gzip允许用户自定义压缩级别、压缩长度阈值和针对请求类型压缩:
	
	1. 压缩级别, `gzipCompressLevel = 9`,取值为1~9,如果不设置为1(最快压缩) 
	
	2. 压缩长度阈值, `gzipMinLength = 256`,当原始内容长度大于此阈值时才开启压缩,默认为20B(ngnix默认长度) 
	
	3. 请求类型, `includedMethods = get;post`,针对哪些请求类型进行压缩,默认只针对GET请求压缩 
	
	


* MaxMemory

	文件上传默认内存缓存大小，默认值是 `1 << 26`(64M)。

	`beego.BConfig.MaxMemory = 1 << 26`

* EnableErrorsShow

	是否显示系统错误信息，默认为 true。

	`beego.BConfig.EnableErrorsShow = true`

#### Web配置

* AutoRender

	是否模板自动渲染，默认值为 true，对于 API 类型的应用，应用需要把该选项设置为 false，不需要渲染模板。

	`beego.BConfig.WebConfig.AutoRender = true`

* EnableDocs

	是否开启文档内置功能，默认是false

	`beego.BConfig.WebConfig.EnableDocs = true`

* FlashName

	Flash数据设置时Cookie的名称，默认是 BEEGO_FLASH

	`beego.BConfig.WebConfig.FlashName = "BEEGO_FLASH"`

* FlashSeperator

	Flash数据的分隔符，默认是 BEEGOFLASH

	`beego.BConfig.WebConfig.FlashSeperator = "BEEGOFLASH"`

* DirectoryIndex

	是否开启静态目录的列表显示，默认不显示目录，返回 403 错误。

	`beego.BConfig.WebConfig.DirectoryIndex = false`

* StaticDir

	静态文件目录设置，默认是static
	
	可配置单个或多个目录:
	1. 单个目录, `StaticDir = download`. 相当于`beego.SetStaticPath("/download","download")`
	
	2. 多个目录, `StaticDir = download:down download2:down2`. 相当于`beego.SetStaticPath("/download","down")`和`beego.SetStaticPath("/download2","down2")`

    `beego.BConfig.WebConfig.StaticDir`

* StaticExtensionsToGzip

	允许哪些后缀名的静态文件进行gzip压缩，默认支持 .css 和 .js

	`beego.BConfig.WebConfig.StaticExtensionsToGzip = []string{".css", ".js"}`

	等价config文件中

		StaticExtensionsToGzip = .css, .js


* TemplateLeft

	模板左标签，默认值是`{{`。

	`beego.BConfig.WebConfig.TemplateLeft="{{"`

* TemplateRight

	模板右标签，默认值是`}}`。

	`beego.BConfig.WebConfig.TemplateRight="}}"`

* ViewsPath

	模板路径，默认值是 views。

	`beego.BConfig.WebConfig.ViewsPath="views"`

* EnableXSRF

	是否开启 XSRF，默认为 false，不开启。

	`beego.BConfig.WebConfig.EnableXSRF = false`

* XSRFKEY

	XSRF 的 key 信息，默认值是 beegoxsrf。 EnableXSRF＝true 才有效

	`beego.BConfig.WebConfig.XSRFKEY = "beegoxsrf"`

* XSRFExpire

	XSRF 过期时间，默认值是 0，不过期。

	`beego.BConfig.WebConfig.XSRFExpire = 0`

#### 监听配置

* Graceful

	是否开启热升级，默认是false，关闭热升级。

	`beego.BConfig.Listen.Graceful=false`

* ServerTimeOut

	设置 HTTP 的超时时间，默认是 0，不超时。

	`beego.BConfig.Listen.ServerTimeOut=0`

* ListenTCP4

	监听本地网络地址类型，默认为 "tcp4"，该值可以是"tcp", "tcp4", "tcp6", "unix" or "unixpacket"之一。

	`beego.BConfig.Listen.ListenTCP4 = "tcp4"`

* EnableHTTP

	是否启用HTTP监听，默认是true。

	`beego.BConfig.Listen.EnableHTTP = true`

* HTTPAddr

	应用监听地址，默认为空，监听所有的网卡 IP。

	`beego.BConfig.Listen.HTTPAddr = ""`

* HTTPPort

	应用监听端口，默认为 8080。

	`beego.BConfig.Listen.HTTPPort = 8080`

* EnableHTTPS

	是否启用 HTTPS，默认是 false 关闭。当需要启用时，先设置 EnableHTTPS = true，并设置`HTTPSCertFile` 和 `HTTPSKeyFile`

	`beego.BConfig.Listen.EnableHTTPS = false`

* HTTPSAddr

	应用监听地址，默认为空，监听所有的网卡 IP。

	`beego.BConfig.Listen.HTTPSAddr = ""`

* HTTPSPort

	应用监听端口，默认为 10443

	`beego.BConfig.Listen.HTTPSPort = 10443`

* HTTPSCertFile

	开启HTTPS后，ssl证书路径，默认为空。

	`beego.BConfig.Listen.HTTPSCertFile = "conf/ssl.crt"`

* HTTPSKeyFile

	开启 HTTPS 之后，SSL证书keyfile 的路径。

	`beego.BConfig.Listen.HTTPSKeyFile = "conf/ssl.key"`

* EnableAdmin

	是否开启进程内监控模块，默认 false 关闭。

	`beego.BConfig.Listen.AdminEnable = false`

* AdminAddr

	监控程序监听的地址，默认值是 localhost 。

	`beego.BConfig.Listen.AdminAddr = "localhost"`

* AdminPort

监控程序监听的地址，默认值是 8088 。

`beego.BConfig.Listen.AdminPort = 8088`

* EnableFcgi

	是否启用 fastcgi ， 默认是 false。

	`beego.BConfig.Listen.EnableFcgi = false`

* EnableStdIo

	通过fastcgi 标准I/O，启用 fastcgi 后才生效，默认 false。

	`beego.BConfig.Listen.EnableStdIo = false`

#### Session配置

* SessionOn

	session 是否开启，默认是 false。

	`beego.BConfig.WebConfig.Session.SessionOn = false`

* SessionProvider

	session 的引擎，默认是 memory，详细参见 `session 模块`。

	`beego.BConfig.WebConfig.Session.SessionProvider = ""`

* SessionName

	存在客户端的 cookie 名称，默认值是 beegosessionID。

	`beego.BConfig.WebConfig.Session.SessionName = "beegosessionID"`

* SessionGCMaxLifetime

	session 过期时间，默认值是 3600 秒。

	`beego.BConfig.WebConfig.Session.SessionGCMaxLifetime = 3600`

* SessionProviderConfig

	 配置信息，根据不同的引擎设置不同的配置信息，详细的配置请看下面的引擎设置，详细参见 [session 模块](zh-CN/module/session.md)

* SessionCookieLifeTime

	session 默认存在客户端的 cookie 的时间，默认值是 3600 秒。

	`beego.BConfig.WebConfig.Session.SessionCookieLifeTime = 3600`

* SessionAutoSetCookie

	是否开启SetCookie, 默认 true 不开启。

	`beego.BConfig.WebConfig.Session.SessionAutoSetCookie = true`


* SessionDomain

session cookie存储域名, 默认空。

`beego.BConfig.WebConfig.Session.SessionDomain = ""`

#### Log配置

	log详细配置，请参见 `logs 模块`。

* AccessLogs

	是否输出日志到Log，默认在prod模式下不会输出日志，默认为 false 不输出日志。，此参数不支持配置文件配置。

	`beego.BConfig.Log.AccessLogs = false`

* FileLineNum

	是否在日志里面显示文件名和输出日志行号，默认 true。此参数不支持配置文件配置。

	`beego.BConfig.Log.FileLineNum = true`

* Outputs

	日志输出配置，参考logs模块，console file 等配置，此参数不支持配置文件配置。

	`beego.BConfig.Log.Outputs = map[string]string{"console": ""}`

	or

	`beego.BConfig.Log.Outputs["console"] = ""`
