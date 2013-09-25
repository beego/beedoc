# Session

Beego has a built-in session module and supports four engines, including memory, file, MySQL and redis. You can implement your own engine based on the interface.

It's easy to use sessions in beego. Use following code in your `main()` function:

	beego.SessionOn = true

Or use configuration file:

	sessionon = true

The following example shows you how to use sessions in Beego:

```go
func (this *MainController) Get() {
	v := this.GetSession("asta")
	if v == nil {
		this.SetSession("asta", int(1))
		this.Data["num"] = 0
	} else {
		this.SetSession("asta", v.(int)+1)
		this.Data["num"] = v.(int)
	}
	this.TplNames = "index.tpl"
}
```

We can see that there are few convenient methods:

- SetSession(name string, value interface{})
- GetSession(name string) interface{}
- DelSession(name string)

There are three kinds of operations for session: set, get, and delete.

Of course you can use following code to customized session logic:

```go
sess := this.StartSession()
defer sess.SessionRelease()
```

The sess object has following methods:

* sess.Set()
* sess.Get()
* sess.Delete()
* sess.SessionID()

However, I recommend you to use SetSession、GetSession、DelSession. These operations helper prevent resource leaks.

There are some arguments you can use in session module:

- SessionOn

	Whether enable session or not, default is false, corresponding arguments in configuration file: sessionon.

- SessionProvider

	Setting session engine, default is memory, other options are file, MySQL and redis, corresponding arguments in configuration file: sessionprovider.

- SessionName

	Setting name of cookies, it saves in users' browser with name beegosessionID, corresponding arguments in configuration file: sessionname.

- SessionGCMaxLifetime

	Setting session expired time, default is 3600 seconds, corresponding arguments in configuration: sessiongcmaxlifetime

- SessionSavePath

	Setting save path or link address of corresponding file, MySQL and redis engines, default is empty, corresponding arguments in configuration file: sessionsavepath

When the SessionProvider is file, SessionSavePath saves file path:

	beego.SessionProvider = "file"
	beego.SessionSavePath = "./tmp"

When the SessionProvider is mysql, SessionSavePath is DSN string, it uses driver [go-sql-driver](https://github.com/go-sql-driver/mysql):

	beego.SessionProvider = "mysql"
	beego.SessionSavePath = "username:password@protocol(address)/dbname?param=value"

When the SessionProvider is redis, SessionSavePath is the address to your redis server, it uses driver [redigo](https://github.com/garyburd/redigo):

	beego.SessionProvider = "redis"
	beego.SessionSavePath = "127.0.0.1:6379"

## Flash

Flash messages in beego are entirely un-related to Adobe Flash. Instead, they are used for transmitting temporary data between two requests, and data saved in flash will be deleted in the next request. Generally, it's used to pass hints and error messages. It fits well with the [Post/Redirect/Get](http://en.wikipedia.org/wiki/Post/Redirect/Get) paradigm.

Let's see an example:

```go
// Show setting information.
func (c *MainController) Get() {
	flash := beego.ReadFromRequest(&c.Controller)
	if n, ok := flash.Data["notice"]; ok {
		// Succeed to show setting information.
		c.TplNames = "set_success.html"
	} else if n, ok = flash.Data["error"]; ok {
		// Show error.
		c.TplNames = "set_error.html"
	} else {
		// Show default setting page.
		this.Data["list"] = GetInfo()
		c.TplNames = "setting_list.html"
	}
}

// Process setting information.
func (c *MainController) Post() {
	flash := beego.NewFlash()
	setting := Settings{}
	valid := Validation{}
	c.ParseForm(&setting)
	if b, err := valid.Valid(setting); err != nil {
		flash.Error("Settings invalid!")
		flash.Store(&c.Controller)
		c.Redirect("/setting", 302)
		return
	} else if b != nil {
		flash.Error("validation err!")
		flash.Store(&c.Controller)
		c.Redirect("/setting", 302)
		return
	}
	saveSetting(setting)
	flash.Notice("Settings saved!")
	flash.Store(&c.Controller)
	c.Redirect("/setting", 302)
}
```

The general logic of in the above example, is as follows:

1. Execute Get method, it shows setting page because of no flash data.
2. Users submit setting information then execute Post method. A new flash is initialized to store error messages, if setting saved successfully, then store the success message.
3. Redirect as a Get request.
4. The Get request retrieves the flash message, then executes the corresponding logic to show corresponding flash data.

The function `ReadFromRequest` implemented data read and write of flash as default, so you can access your flash data as follows:

	{{.flash.error}}
	{{.flash.warning}}
	{{.flash.notice}}

flash objects have 3 levels:

- Notice
- Warning
- Error
