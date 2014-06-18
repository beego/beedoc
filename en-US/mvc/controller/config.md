---
name: Configuration
sort: 1
---

# Configuration

Beego configuration file supports INI, XML, JSON, YAML. It's using INI format by default. It is flexible and easy to configure.

## The default configurations parsing 

Beego will parse `conf/app.conf` file by default.

You can initialize many beego default variables in this file:

	appname = beepkg
	httpaddr = "127.0.0.1"
	httpport = 9090
	runmode ="dev"
	autorender = false
	autorecover = false
	viewspath = "myview"

Thess configurations will replace beego's default value.

You can also configure many values which your application needed, such as database connection:

	mysqluser = "root"
	mysqlpass = "rootpass"
	mysqlurls = "127.0.0.1"
	mysqldb   = "beego"

Then you can read these configurations like this:

	beego.AppConfig.String("mysqluser")
	beego.AppConfig.String("mysqlpass")
	beego.AppConfig.String("mysqlurls")
	beego.AppConfig.String("mysqldb")

AppConfig supports:

- Bool(key string) (bool, error)
- Int(key string) (int, error)
- Int64(key string) (int64, error)
- Float(key string) (float64, error)
- String(key string) string

### The Configurations for Different Environments

You can set configurations for different Runmode under their own sections. Beego will take the configurations of current Runmode by default. For example:

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

### Beego default variables

Beego has many configurable variables. Let's have a look of this variables. It will help us to know how to use them in development. (You can configure and overwrite them in `conf/app.conf`. Case insensitive.):

* AppName
  
  Application name, Beego by default. It's project_name if the application is created by `bee new project_name`

* AppPath

  Application path. It will get the first parameter of the executed command by `os.Args[0]`. So you need to execute by full path if you are using supervisor to manage processes.
	
* AppConfigPath

  Application configuration file path. it's `conf/app.conf` by default.  You can change it to your own file.

* EnableHttpListen

  Enable http listen or not, enabled by default.

* HttpAddr

  Application listening address, empty by default which will listen all network adapter's IPs.

* HttpPort

  Application listening port, 8080 by default.
	
* EnableHttpTLS

  Enable https or not, disabled by default.

* HttpsPort

  Application listening https port, 10443 by default.

* HttpCertFile

  If https is enabled, the path of certfile.

* HttpKeyFile
		
  If https is enabled, the path of keyfile.

* HttpServerTimeOut

  Config the http timeout, 0 by default which means no timeout.
	
* RunMode

  The application mode, dev by default. In dev mode it will show user friendly error pages as we saw before.

* AutoRender

  Use auto render or not, true by default. Should set it to false for API application, no need to render template.

* RecoverPanic

  Recover from panic or not, true by default. It will recover from exceptions without exiting application.

* ViewsPath

  The path of templates, views by default.

* SessionOn

  Enable session or not, false by default.

* SessionProvider

  Session provider, memory by default.

* SessionName

  The session cookie name stored in browser. beegosessionID by default.

* SessionGCMaxLifetime

  Valide time of session, 3600s by default.

* SessionSavePath

  Session save path, empty by default.

* SessionHashFunc

  Function that generate sessionID, sha1 by default.

* SessionHashKey

  Hash key of session.
	
* SessionCookieLifeTime

  The valid time of cookie in browser for seesion, 3600s by default.

* UseFcgi

  Enable fastcgi or not, false by default.

* MaxMemory

  Memory cache size for file uploading, `1 << 26`(64M) by default.

* EnableGzip

  Enable Gzip or not, false by default. If Gzip is enabled, the output of template will be compressed by Gzip or zlib according to `Accept-Encoding` of browser.

* DirectoryIndex

  Enable list static directory or not, disabled by default. It will return 403 error.
	
* BeegoServerName

  Beego server will output `beego` as server name.
	
* EnableAdmin

  Enable supervisor module or not, disabled by default.
	
* AdminHttpAddr

  Listening address of supervisor, `localhost` by default.

* AdminHttpPort
  Listening port of supervisor, 8088 by default.
	
* TemplateLeft

  Left mark of template, `{{` by default.
	
* TemplateRight

  Right mark of template, `}}` by default.
	
* ErrorsShow

  Show error or not, show by default.

* EnableXSRF
  Enable XSRF

* XSRFKEY

  XSRF key, beegoxsrf by default.
	
* XSRFExpire

  XSRF expire time, 0 by default.

* FlashName

  Flash Cookie name，default is `BEEGO_FLASH`

* FlashSeperator

  Flash data seperator，default is `BEEGOFLASH`

* StaticDir

  set the static file，default is `static`	
 	1. one dir，the same as `beego.SetStaticPath("/download","download")`

		StaticDir = download
  	2. multi dirs, the same as `beego.SetStaticPath("/download","down")` and `beego.SetStaticPath("/download2","down2")`

    StaticDir = download:down download2:down2 
    
* EnableDocs

  Enable Docs or not, default is `false`
  
* AppConfigProvider

  File format of AppConfg, default is `ini`. Couble be `xml`, `yaml`, `json` as well.
