---
name: Configuration
sort: 1
---

# Configuration

By default the Beego configuration file uses the INI format.  Other supported formats include XML, JSON, and YAML.

## Default configurations parsing

Beego will parse the `conf/app.conf` file by default.

Many default variables can be initialized in this file:

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090
	runmode ="dev"
	autorender = false
	recoverpanic = false
	viewspath = "myview"

These configurations will replace Beego's default values.

Other application specific values can also be set using this file, such as database connection details:

	mysqluser = "root"
	mysqlpass = "rootpass"
	mysqlurls = "127.0.0.1"
	mysqldb   = "beego"

These configurations can be accessed like this:

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

When using the INI format the key supports the section::key pattern.
 
The Default* methods can be used to return default values if the config file cannot be read.


### Configurations for Different Environments

Configurations for different runmodes can be set under their own sections. Beego will take the configurations of the current runmode by default. For example:

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

The configurations above set up httpport for dev, prod and test environments. Beego will take httpport = 8080 for the current runmode "dev".

To get config to operate under a different runmode use "runmode::key". For example:

`beego.AppConfig.String("dev::mysqluser")`

For custom configs use `beego.GetConfig(typ, key string)` to get the config.


### Multiple config files
The INI config file supports `include` to including multiple config files.

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



### Beego default config variables

Beego includes many configurable variables. These can be configured and overwritten in `conf/app.conf`.

#### Basic config

* AppConfigPath

    The application configuration file path. By default this is `conf/app.conf`.  
  
    `beego.AppConfigPath = "conf/app2.conf"`

* AppConfigProvider

    The file format of AppConfig. By default this is `ini`. Other valid formats include `xml`, `yaml`, and `json`.

	`beego.AppConfigProvider = "ini"`

#### App config

* AppName

    The application name. By default this is "Beego".  If the application is created by `bee new project_name` it will be set to project_name. 
  
  	`beego.BConfig.AppName = "beego"`
  	
* RunMode

    The application mode. By default this is set to `dev`. Other valid modes include `prod` and `test`.  In `dev` mode user friendly error pages will be shown.  In `prod` mode user friendly error pages will not be rendered.
    
    `beego.BConfig.RunMode = "dev"`
 
* RouterCaseSensitive

    Set case sensitivity for the router.  By default this value is true.
    
    `beego.BConfig.RouterCaseSensitive = true`
   
* ServerName

    The Beego server name.  By defalt this name is `beego`.

	`beego.BConfig.ServerName = "beego"`
	
* RecoverPanic

    When active the application will recover from exceptions without exiting the application.  By default this is set to true.  
    
    `beego.BConfig.RecoverPanic = true`

* CopyRequestBody

    Toggle copying of raw request body in context. By default this is false except for GET, HEAD or file uploading.
  
  	`beego.BConfig.CopyRequestBody = false`

* EnableGzip

    Enable Gzip.  By default this is false. If Gzip is enabled the output of templates will be compressed by Gzip or zlib according to the `Accept-Encoding` setting of the browser.
  
    `beego.BConfig.EnableGzip = false`
    
    Further properties can be configured as below:
    
    `gzipCompressLevel = 9` Sets the compression level used for deflate compression(0-9).  By default is 9 (best speed). 
    
    `gzipMinLength = 256` Original content will only be compressed if length is either unknown or greater than gzipMinLength. The default length is 20B. 
    
    `includedMethods = get;post` List of HTTP methods to compress. By default only GET requests are compressed.

* MaxMemory

    Sets the memory cache size for file uploading.  By default this is `1 << 26`(64M).
    
    `beego.BConfig.MaxMemory = 1 << 26`

* EnableErrorsShow

    Toggles the display of error messages.  By default this is True.

	`beego.BConfig.EnableErrorsShow = true`

* EnableErrorsRender

	Toggles rendering error messages. By default this is set to True.  User friendly error pages will not be rendered even in dev `RunMode` if this value is false.

#### Web config

* AutoRender

    Enable auto render.  By default this is True.  This value should be set to false for API applications, as there is no need to render templates.
  
    `beego.BConfig.WebConfig.AutoRender = true`
    
* EnableDocs

    Enable Docs. By default this is False.

	`beego.BConfig.WebConfig.EnableDocs = false`

* FlashName

    Sets the Flash Cookie name.  By default this is `BEEGO_FLASH`.
    
  	`beego.BConfig.WebConfig.FlashName = "BEEGO_FLASH"`

* FlashSeperator

    Set the Flash data separator.  By default this is `BEEGOFLASH`.
  
  	`beego.BConfig.WebConfig.FlashSeperator = "BEEGOFLASH"`

* DirectoryIndex

    Enable listing of the static directory. By defalt this is False and will return a 403 error.

	`beego.BConfig.WebConfig.DirectoryIndex = false`

* StaticDir

    Sets the static file dir(s).  By default this is `static`.
    
 	1. Single dir, `StaticDir = download`. Same as `beego.SetStaticPath("/download","download")`

  	2. Multiple dirs, `StaticDir = download:down download2:down2`. Same as `beego.SetStaticPath("/download","down")` and `beego.SetStaticPath("/download2","down2")`

    `beego.BConfig.WebConfig.StaticDir = map[string]string{"download":"download"}`


* StaticExtensionsToGzip

    Sets a list of file extensions that will support compression by Gzip. The formats `.css` and `.js` are supported by default.
    
	`beego.BConfig.WebConfig.StaticExtensionsToGzip = []string{".css", ".js"}`
	
    Same as in config file StaticExtensionsToGzip = .css, .js

* TemplateLeft

    Left mark of the template, `{{` by default.

	`beego.BConfig.WebConfig.TemplateLeft = "{{"`

* TemplateRight

    Right mark of the template, `}}` by default.
    
    `beego.BConfig.WebConfig.TemplateRight = "}}"`

* ViewsPath

    Set the location of template files. This is set to `views` by default.
    
	`beego.BConfig.WebConfig.ViewsPath = "views"`

* EnableXSRF
    Enable XSRF

	`beego.BConfig.WebConfig.EnableXSRF = false`
	
* XSRFKEY

    Set the XSRF key.  By default this is `beegoxsrf`.

	`beego.BConfig.WebConfig.XSRFKEY = "beegoxsrf"`
	
* XSRFExpire

    Set the XSRF expire time. By default this is set to `0`.

	`beego.BConfig.WebConfig.XSRFExpire = 0`

#### HTTP Server config

* Graceful

    Enable graceful shutdown. By default this is False.
  
  	`beego.BConfig.Listen.Graceful = false`
  
* ServerTimeOut

    Set the http timeout. By default thi is '0', no timeout.

	`beego.BConfig.Listen.ServerTimeOut = 0`

* ListenTCP4

    Set the address type. default is `tcp6` but we can set it to true to force use `TCP4`.

	`beego.BConfig.Listen.ListenTCP4 = true`

* EnableHTTP

	Enable HTTP listen. By default this is set to True.

	`beego.BConfig.Listen.EnableHTTP = true`

* HTTPAddr

	Set the address the app listens to. By default this value is empty and the app will listen to all IPs.

	`beego.BConfig.Listen.HTTPAddr = ""`

* HTTPPort

    Set the port the app listens on. By default this is 8080

	`beego.BConfig.Listen.HTTPPort = 8080`

* EnableHTTPS

    Enable HTTPS. By default this is False. When enabled `HTTPSCertFile` and `HTTPSKeyFile` must also be set.

	`beego.BConfig.Listen.EnableHTTPS = false`

* HTTPSAddr

	Set the address the app listens to. Default is empty and the app will listen to all IPs.

	`beego.BConfig.Listen.HTTPSAddr = ""`

* HTTPSPort

    Set the port the app listens on. By default this is 10443

	`beego.BConfig.Listen.HTTPSPort = 10443`

* HTTPSCertFile

	Set the SSL cert path. By default this value is empty.

	`beego.BConfig.Listen.HTTPSCertFile = "conf/ssl.crt"`

* HTTPSKeyFile

	Set the SSL key path. By default this value is empty.

	`beego.BConfig.Listen.HTTPSKeyFile = "conf/ssl.key"`

* EnableAdmin

    Enable supervisor module.  By default this is False.

	`beego.BConfig.Listen.EnableAdmin = false`

* AdminAddr

    Set the address the admin app listens to. By defalt this is blank and the app will listen to any IP.
    
    `beego.BConfig.Listen.AdminAddr = ""`

* AdminPort

    Set the port the admin app listens on. By default this is 8088.
    
	`beego.BConfig.Listen.AdminPort = 8088`

* EnableFcgi

    Enable fastcgi. By default this is False.
    
    `beego.BConfig.Listen.EnableFcgi = false`

* EnableStdIo

	Enable fastcgi standard I/O or not. By default this is False.

	`beego.BConfig.Listen.EnableStdIo = false`

#### Session config

* SessionOn

    Enable session. By defalt this is False.
    
	`beego.BConfig.WebConfig.Session.SessionOn = false`

* SessionProvider

    Set the session provider.  By default this is `memory`.
    
    `beego.BConfig.WebConfig.Session.SessionProvider = "memory"`

* SessionName

    Set the session cookie name stored in the browser.  By default this is `beegosessionID`.
    
  	`beego.BConfig.WebConfig.Session.SessionName = "beegosessionID"`
  	
* SessionGCMaxLifetime

    Set the session expire time.  By default this is 3600s.
  
  	`beego.BConfig.WebConfig.Session.SessionGCMaxLifetime = 3600`
  	
* SessionProviderConfig

    Set the session provider config. Different providers can require different config settings. Please see [session](en-US/module/session.md) for more information.

* SessionCookieLifeTime

    Set the valid expiry time of the cookie in browser for session.  By default this is 3600s.

	`beego.BConfig.WebConfig.Session.SessionCookieLifeTime = 3600`

* SessionAutoSetCookie

	Enable SetCookie. By default this is True.

	`beego.BConfig.WebConfig.Session.SessionAutoSetCookie = true`

* SessionDomain

	Set the session cookie domain. By default this is empty.

	`beego.BConfig.WebConfig.Session.SessionDomain = ""`

#### Log config

	See [logs module](en-US/module/logs.md) for more information.
 
* AccessLogs

    Enable output access logs. By default these logs will not be output under 'prod' mode.

	`beego.BConfig.Log.AccessLogs = false`

* FileLineNum

    Toggle printing line numbers. By default this is True.  This config is not supported in config file.

	`beego.BConfig.Log.FileLineNum = true`

* Outputs

    Log outputs config. This config is not supported in config file.

	`beego.BConfig.Log.Outputs = map[string]string{"console": ""}`

	or

	`beego.BConfig.Log.Outputs["console"] = ""`

