# Session/Flash

Beego has a built-in session module and supports four engines, including memory, file, MySQL and redis. You can implement your own engine based on the interface.

It's easy to use session in Beego, use following code in your main() function:

	beego.SessionOn = true

Or use configuration file:

	sessionon = true

The following example shows you how to use session in Beego:

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

We can see that there are few convenient methods:

- SetSession(name string, value interface{})
- GetSession(name string) interface{}
- DelSession(name string)

There are three kinds of operation for session: set, get, and delete.

Of course you can use following code to customized session logic:

	sess:=this.StartSession()
	defer sess.SessionRelease()

The sess object has following methods:

* sess.Set()
* sess.Get()
* sess.Delete()
* sess.SessionID()

However, I recommend you to use SetSession、GetSession、DelSession these three operations in order to prevent resource leak.

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

When the SessionProvider is mysql, SessionSavePath is link address, it uses driver [go-sql-driver](https://github.com/go-sql-driver/mysql):

	beego.SessionProvider = "mysql"
	beego.SessionSavePath = "username:password@protocol(address)/dbname?param=value"

When the SessionProvider is redis, SessionSavePath is link address of redis, it uses driver [redigo](https://github.com/garyburd/redigo):

	beego.SessionProvider = "redis"
	beego.SessionSavePath = "127.0.0.1:6379"