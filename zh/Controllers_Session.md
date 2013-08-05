# Session

Beego 内置了 session 模块，目前 session 模块支持的后端引擎包括 memory、file、mysql、redis 四种，用户也可以根据相应的 interface 实现自己的引擎。

Beego 中使用 session 相当方便，只要在 main 入口函数中设置如下：

	beego.SessionOn = true

或者通过配置文件配置如下：

	sessionon = true

通过这种方式就可以开启 session，如何使用 session，请看下面的例子：

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

session 有几个方便的方法：

- SetSession(name string, value interface{})
- GetSession(name string) interface{}
- DelSession(name string)

session 操作主要有设置 session、获取 session、删除 session。

当然你要可以通过下面的方式自己控制相应的逻辑这些逻辑：

	sess:=this.StartSession()
	defer sess.SessionRelease()

sess对象具有如下方法：

* sess.Set()
* sess.Get()
* sess.Delete()
* sess.SessionID()

但是我还是建议大家采用 SetSession、GetSession、DelSession 三个方法来操作，避免自己在操作的过程中资源没释放的问题。

关于 Session 模块使用中的一些参数设置：

- SessionOn

	设置是否开启 Session，默认是 false，配置文件对应的参数名：sessionon。

- SessionProvider

	设置 Session 的引擎，默认是 memory，目前支持还有 file、mysql、redis 等，配置文件对应的参数名：sessionprovider。

- SessionName

	设置 cookies 的名字，Session 默认是保存在用户的浏览器 cookies 里面的，默认名是 beegosessionID，配置文件对应的参数名是：sessionname。

- SessionGCMaxLifetime

	设置 Session 过期的时间，默认值是 3600 秒，配置文件对应的参数：sessiongcmaxlifetime。

- SessionSavePath

	设置对应 file、mysql、redis 引擎的保存路径或者链接地址，默认值是空，配置文件对应的参数：sessionsavepath。


当 SessionProvider 为 file 时，SessionSavePath 是只保存文件的目录，如下所示：

	beego.SessionProvider = "file"
	beego.SessionSavePath = "./tmp"

当 SessionProvider 为 mysql 时，SessionSavePath 是链接地址，采用 [go-sql-driver](https://github.com/go-sql-driver/mysql)，如下所示：

	beego.SessionProvider = "mysql"
	beego.SessionSavePath = "username:password@protocol(address)/dbname?param=value"

当 SessionProvider 为 redis 时，SessionSavePath 是 redis 的链接地址，采用了 [redigo](https://github.com/garyburd/redigo)，如下所示：

	beego.SessionProvider = "redis"
	beego.SessionSavePath = "127.0.0.1:6379"

## Flash
这个 flash 与 Adobe/Macromedia Flash 没有任何关系。它主要用于在两个逻辑间传递临时数据，flash 中存放的所有数据会在紧接着的下一个逻辑中调用后清除。一般用于传递提示和错误消息。它适合 [Post/Redirect/Get](http://en.wikipedia.org/wiki/Post/Redirect/Get) 模式。下面看使用的例子：

	// 显示设置信息
	func (c *MainController) Get() {
		flash:=beego.ReadFromRequest(c)
		if n,ok:=flash.Data["notice"];ok{
			//显示设置成功
			c.TplNames = "set_success.html"
		}else if n,ok=flash.Data["error"];ok{
			//显示错误
			c.TplNames = "set_error.html"
		}else{
			// 不然默认显示设置页面
			this.Data["list"]=GetInfo()
			c.TplNames = "setting_list.html"
		}
	}
	
	// 处理设置信息
	func (c *MainController) Post() {
		flash:=beego.NewFlash()
		setting:=Settings{}
		valid := Validation{}
		c.ParseForm(&setting)
		if b, err := valid.Valid(setting);err!=nil {
			flash.Error("Settings invalid!")
			flash.Store(c)
			c.Redirect("/setting",302)
			return
		}else if b!=nil{
			flash.Error("validation err!")
			flash.Store(c)
			c.Redirect("/setting",302)
			return
		}	
		saveSetting(setting)
		flash.Notice("Settings saved!")
		flash.Store(c)
		c.Redirect("/setting",302)
	}

上面的代码执行的大概逻辑是这样的：

1. Get 方法执行，因为没有 flash 数据，所以显示设置页面。
2. 用户设置信息之后点击递交，执行 Post，然后初始化一个 flash，通过验证，验证出错或者验证不通过设置 flash 的错误，如果通过了就保存设置，然后设置 flash 成功设置的信息。
3. 设置完成后跳转到 Get 请求。
4. Get 请求获取到了 Flash 信息，然后执行相应的逻辑，如果出错显示出错的页面，如果成功显示成功的页面。

默认情况下 `ReadFromRequest` 函数已经实现了读取的数据赋值给 flash，所以在你的模板里面你可以这样读取数据：

	{{.flash.error}}
	{{.flash.warning}}
	{{.flash.notice}}
	
flash 对象有三个级别的设置：

* Notice 提示信息
* Warning 警告信息
* Error 错误信息