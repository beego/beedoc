---
name: Configuration
sort: 1
---

# Configuration

Beego configuration file supports INI, XML, JSON, YAML. It uses INI format by default. It is flexible and easy to configure.

## The default configurations parsing

Beego will parse `conf/app.conf` file by default.

You can initialize many Beego default variables in this file:

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090
	runmode ="dev"
	autorender = false
	autorecover = false
	viewspath = "myview"

These configurations will replace Beego's default value.

You can also configure many values which your application needs, such as database connection:

	mysqluser = "root"
	mysqlpass = "rootpass"
	mysqlurls = "127.0.0.1"
	mysqldb   = "beego"

Then you can read these configurations like this:

	beego.AppConfig.String("mysqluser")
	beego.AppConfig.String("mysqlpass")
	beego.AppConfig.String("mysqlurls")
	beego.AppConfig.String("mysqldb")

AppConfig's methods:

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

The key supports section::key when using ini type.
 
You can use Default* methods to return default value if can't read from config file.


### The Configurations for Different Environments

You can set configurations for different runmode under their own sections. Beego will take the configurations of current runmode by default. For example:

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

The configurations above set up httpport for dev, prod and test environment. Beego will take httpport = 8080 for current runmode "dev".

To get the config under different runmode, you can use "runmode::key". Such as `beego.AppConfig.String("dev::mysqluser")`

For custom configs, you need to use `beego.GetConfig(typ, key string)` to get the config. (1.4.0+).


### Multiple config files
INI config file supports `include` to including multiple config files.

app.conf

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090

	include "app2.conf"

app2.conf

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



### Beego default config variables

Beego has many configurable variables. Let's have a look at these variables. It will help us to know how to use them in development. (You can configure and overwrite them in `conf/app.conf`. Case insensitive.):

#### Basic config

* AppConfigPath

    Application configuration file path. It is `conf/app.conf` by default.  You can change it to your own file.
  
    `beego.AppConfigPath = "conf/app2.conf"`

* AppConfigProvider

    File format of AppConfig, default is `ini`. Could be `xml`, `yaml`, `json` as well.

	`beego.AppConfigProvider = "ini"`

#### App config

* AppName

    Application name, Beego by default. It's project_name if the application is created by `bee new project_name`
  
  	`beego.BConfig.AppName = "beego"`
  	
* RunMode

    The application mode, which can be `dev`, `prod` or `test`, `dev` by default. In `dev` mode it will show user friendly error pages as we saw before but it won't show in `prod` mode.
    
    `beego.BConfig.RunMode = "dev"`
 
* RouterCaseSensitive

    Set the router case sensitivity or not. Default value is true.
    
    `beego.BConfig.RouterCaseSensitive = true`
   
* ServerName

    Beego server will output `beego` as server name by default.

	`beego.BConfig.ServerName = "beego"`
	
* RecoverPanic

    Recover from panic or not, true by default. It will recover from exceptions without exiting application.
    
    `beego.BConfig.RecoverPanic = true`

* CopyRequestBody

    Flag of copy raw request body in context, true by default except GET, HEAD or file uploading.
  
  	`beego.BConfig.CopyRequestBody = true`

* EnableGzip

    Enable Gzip or not, false by default. If Gzip is enabled, the output of template will be compressed by Gzip or zlib according to `Accept-Encoding` of browser.
  
    `beego.BConfig.EnableGzip = false`

* MaxMemory

    Memory cache size for file uploading, `1 << 26`(64M) by default.
    
    `beego.BConfig.MaxMemory = 1 << 26`

* EnableErrorsShow

    Whether show error messages or not. True by default.

	`beego.BConfig.EnableErrorsShow = true`

#### Web config

* AutoRender

    Use auto render or not, true by default. Should set it to false for API application, no need to render template.
  
    `beego.BConfig.WebConfig.AutoRender = true`
    
* EnableDocs

    Enable Docs or not, default is `false`

	`beego.BConfig.WebConfig.EnableDocs = true`

* FlashName

    Flash Cookie name，default is `BEEGO_FLASH`
    
  	`beego.BConfig.WebConfig.FlashName = "BEEGO_FLASH"`

* FlashSeperator

    Flash data seperator，default is `BEEGOFLASH`
  
  	`beego.BConfig.WebConfig.FlashSeperator = "BEEGOFLASH"`

* DirectoryIndex

    Enable list static directory or not, disabled by default. It will return 403 error.

	`beego.BConfig.WebConfig.DirectoryIndex = false`

* StaticDir

    Set the static file dir(s), default is `static`
    
 	1. Single dir, `StaticDir = download`. Same as `beego.SetStaticPath("/download","download")`

  	2. Multiple dirs, `StaticDir = download:down download2:down2`. Same as `beego.SetStaticPath("/download","down")` and `beego.SetStaticPath("/download2","down2")`

    `beego.BConfig.WebConfig.StaticDir = map[string]string{"download":"download"}`


* StaticExtensionsToGzip

    Set a list of file extensions. The static file with the extension will supports gzip compress. It supports `.css` and `.js` by default.
    
	`beego.BConfig.WebConfig.StaticExtensionsToGzip = []string{".css", ".js"}`
	
    Same as in config file StaticExtensionsToGzip = .css, .js

* TemplateLeft

    Left mark of template, `{{` by default.

	`beego.BConfig.WebConfig.TemplateLeft="{{"`

* TemplateRight

    Right mark of template, `}}` by default.
    
    `beego.BConfig.WebConfig.TemplateRight="}}"`

* ViewsPath

    The path of templates, views by default.
    
	`beego.BConfig.WebConfig.ViewsPath="views"`

* EnableXSRF
    Enable XSRF

	`beego.BConfig.WebConfig.EnableXSRF = false`
	
* XSRFKEY

    XSRF key, beegoxsrf by default.

	`beego.BConfig.WebConfig.XSRFKEY = "beegoxsrf"`
	
* XSRFExpire

    XSRF expire time, 0 by default.

	`beego.BConfig.WebConfig.XSRFExpire = 0`

#### HTTP Server config

* Graceful

    Enable graceful shutdown or not. Default is disabled.
  
  	`beego.BConfig.Listen.Graceful=false`
  
* ServerTimeOut

    Set the http timeout. Default is 0, no timeout.

	`beego.BConfig.Listen.ServerTimeOut=0`

* ListenTCP4

    The address type. Default is "tcp4". The value can be "tcp", "tcp4", "tcp6", "unix" or "unixpacket"

	`beego.BConfig.Listen.ListenTCP4 = "tcp4"`

* EnableHTTP

	Weather enable HTTP listen. Default is true.

	`beego.BConfig.Listen.EnableHTTP = true`

* HTTPAddr

	The address app listens to. Default is empty, listen all IPs.

	`beego.BConfig.Listen.HTTPAddr = ""`

* HTTPPort

    The port the app listens to. Default is 8080

	`beego.BConfig.Listen.HTTPPort = 8080`

* EnableHTTPS

    Whether enable HTTPS or not. Default is false. To enable, set `EnableHTTPListen = false` and set `HTTPSCertFile` and `HTTPSKeyFile`

	`beego.BConfig.Listen.EnableHTTPS = false`

* HTTPSAddr

	The address app listens to. Default is empty, listen all IPs.

	`beego.BConfig.Listen.HTTPSAddr = ""`

* HTTPSPort

    The port the app listens to. Default is 10443

	`beego.BConfig.Listen.HTTPPort = 10443`

* HTTPSCertFile

	SSL cert path. Default is empty.

	`beego.BConfig.Listen.HTTPSCertFile = "conf/ssl.crt"`

* HTTPSKeyFile

	SSL key path. Default is empty.

	`beego.BConfig.Listen.HTTPSKeyFile = "conf/ssl.key"`

* EnableAdmin

    Enable supervisor module or not, disabled by default.

	`beego.BConfig.Listen.AdminEnable = false`

* AdminAddr

    The address admin app listens to. Default is "localhost".
    
    `beego.BConfig.Listen.AdminAddr = "localhost"`

* AdminPort

    The port the admin app listens to. Default is 8088.
    
	`beego.BConfig.Listen.AdminPort = 8088`

* EnableFcgi

    Whether enable fastcgi or not. Default is false 
    
    `beego.BConfig.Listen.EnableFcgi = false`

* EnableStdIo

	Whether use fastcgi standard I/O. Default is false

	`beego.BConfig.Listen.EnableStdIo = false`

#### Session config

* SessionOn

    Enable session or not, false by default.
    
	`beego.BConfig.WebConfig.Session.SessionOn = false`

* SessionProvider

    Session provider, memory by default.
    
    `beego.BConfig.WebConfig.Session.SessionProvider = ""`

* SessionName

    The session cookie name stored in browser. beegosessionID by default.
    
  	`beego.BConfig.WebConfig.Session.SessionName = "beegosessionID"`
  	
* SessionGCMaxLifetime

    Session expire time, 3600s by default.
  
  	`beego.BConfig.WebConfig.Session.SessionGCMaxLifetime = 3600`
  	
* SessionProviderConfig

    The session provider config. Different config for different provider. Please check [session](en-US/module/session.md).

* SessionCookieLifeTime

    The valid time of cookie in browser for session, 3600s by default.

	`beego.BConfig.WebConfig.Session.SessionCookieLifeTime = 3600`

* SessionAutoSetCookie

	Enable SetCookie or not. Default is true.

	`beego.BConfig.WebConfig.Session.SessionAutoSetCookie = true`

* SessionDomain

session cookie domain. Default is empty.

`beego.BConfig.WebConfig.Session.SessionDomain = ""`

#### Log config

	For detail, see [logs module](en-US/module/logs.md)
 
* AccessLogs

    Output access logs or not.  It won't output access logs under prod mode by default.

	`beego.BConfig.Log.AccessLogs = false`

* FileLineNum

    Whether print line number or not. Default is true. This config is not supported in config file.

	`beego.BConfig.Log.FileLineNum = true`

* Outputs

    Log outputs config. This config is not supported in config file.

	`beego.BConfig.Log.Outputs = map[string]string{"console": ""}`

	or

	`beego.BConfig.Log.Outputs["console"] = ""`

