# 控制器（Controllers）

基于 beego 的 Controller 设计，只需要匿名组合 `beego.Controller` 就可以了，如下所示：

	type xxxController struct {
		beego.Controller
	}

`beego.Controller` 实现了接口 `beego.ControllerInterface`，`beego.ControllerInterface` 定义了如下函数：

- Init(ct *Context, cn string)

	这个函数主要初始化了 Context、相应的 Controller 名称，模板名，初始化模板参数的容器 Data。

- Prepare()

	这个函数主要是为了用户扩展用的，这个函数会在下面定义的这些 Method 方法之前执行，用户可以重写这个函数实现类似用户验证之类。

- Get()

	如果用户请求的 HTTP Method 是 GET，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Get 请求。

- Post()

	如果用户请求的 HTTP Method 是 POST，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Post 请求。

- Delete()

	如果用户请求的 HTTP Method 是 DELETE，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Delete 请求。

- Put()

	如果用户请求的 HTTP Method 是 PUT，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Put 请求.

- Head()

	如果用户请求的 HTTP Method 是 HEAD，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Head 请求。

- Patch()

	如果用户请求的 HTTP Method 是 PATCH，那么就执行该函数，默认是 403，用户继承的子struct中可以实现了该方法以处理Patch请求.

- Options()

	如果用户请求的HTTP Method是OPTIONS，那么就执行该函数，默认是 403，用户继承的子 struct 中可以实现了该方法以处理 Options 请求。

- Finish()

	这个函数实在执行完相应的 HTTP Method 方法之后执行的，默认是空，用户可以在子 strcut 中重写这个函数，执行例如数据库关闭，清理数据之类的工作。

- Render() error

	这个函数主要用来实现渲染模板，如果 beego.AutoRender 为 true 的情况下才会执行。

所以通过子 struct 的方法重写，用户就可以实现自己的逻辑，接下来我们看一个实际的例子：

	type AddController struct {
		beego.Controller
	}

	func (this *AddController) Prepare() {

	}

	func (this *AddController) Get() {
		this.Data["content"] = "value"
		this.Layout = "admin/layout.html"
		this.TplNames = "admin/add.tpl"
	}

	func (this *AddController) Post() {
		pkgname := this.GetString("pkgname")
		content := this.GetString("content")
		pk := models.GetCruPkg(pkgname)
		if pk.Id == 0 {
			var pp models.PkgEntity
			pp.Pid = 0
			pp.Pathname = pkgname
			pp.Intro = pkgname
			models.InsertPkg(pp)
			pk = models.GetCruPkg(pkgname)
		}
		var at models.Article
		at.Pkgid = pk.Id
		at.Content = content
		models.InsertArticle(at)
		this.Ctx.Redirect(302, "/admin/index")
	}

### request 处理

我们经常需要获取用户传递的数据，包括 Get、POST 等方式的请求，Beego 里面会自动解析这些数据，你可以通过如下方式获取数据：

- GetString(key string) string
- GetInt(key string) (int64, error)
- GetBool(key string) (bool, error)

使用例子如下：

	func (this *MainController) Post() {
		jsoninfo := this.GetString("jsoninfo")
		if jsoninfo == "" {
			this.Ctx.WriteString("jsoninfo is empty")
			return
		}
	}

如果你需要的数据可能是其他类型的，例如是 int 类型而不是 int64，那么你需要这样处理：

	func (this *MainController) Post() {
		id := this.Input().Get("id")
		intid, err := strconv.Atoi(id)
	}

更多其他的 request 的信息，用户可以通过 `this.Ctx.Request` 获取信息，关于该对象的属性和方法参考手册 [Request](http://gowalker.org/pkg/net/http#Request)。

### 文件上传

在 Beego 中你可以很容易的处理文件上传，就是别忘记在你的 form 表单中增加这个属性 `enctype="multipart/form-data"`，否者你的浏览器不会传输你的上传文件。

文件上传之后一般是放在系统的内存里面，如果文件的 size 大于设置的缓存内存大小，那么就放在临时文件中，默认的缓存内存是 64M，你可以通过如下来调整这个缓存内存大小:

	beego.MaxMemory = 1<<22

或者在配置文件中通过如下设置：

	maxmemory = 1<<22

Beego 提供了两个很方便的方法来处理文件上传：

- GetFile(key string) (multipart.File, *multipart.FileHeader, error)

	该方法主要用于用户读取表单中的文件名 `the_file`，然后返回相应的信息，用户根据这些变量来处理文件上传：过滤、保存文件等。

- SaveToFile(fromfile, tofile string) error

	该方法是在 GetFile 的基础上实现了快速保存的功能

保存的代码例子如下：

	func (this *MainController) Post() {
		this.SaveToFile("the_file","/var/www/uploads/uploaded_file.txt")
	}


### JSON 和 XML 输出

Beego 当初设计的时候就考虑了 API 功能的设计，而我们在设计 API 的时候经常是输出 JSON 或者 XML 数据，那么 Beego 提供了这样的方式直接输出：

JSON 数据直接输出，设置 `content-type` 为 `application/json`：

	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["json"] = &mystruct
		this.ServeJson()
	}

XML 数据直接输出，设置 `content-type` 为 `application/xml`：

	func (this *AddController) Get() {
		mystruct := { ... }
		this.Data["xml"]=&mystruct
		this.ServeXml()
	}


## 跳转和错误

我们在做 Web 开发的时候，经常会遇到页面调整和错误处理，Beego 这这方面也进行了考虑，通过 `Redirect` 方法来进行跳转：

	func (this *AddController) Get() {
		this.Redirect("/", 302)
	}

如何中止此次请求并抛出异常，Beego 可以在控制器中这操作：

	func (this *MainController) Get() {
		this.Abort("401")
		v := this.GetSession("asta")
		if v == nil {
			this.SetSession("asta", int(1))
			this.Data["Email"] = 0
		} else {
			this.SetSession("asta", v.(int)+1)
			this.Data["Email"] = v.(int)
		}
		this.TplNames = "index.tpl"
	}

这样 `this.Abort("401")` 之后的代码不会再执行，而且会默认显示给用户如下页面：

![](https://raw.github.com/astaxie/beego/master/docs/zh/images/401.png)

Beego 框架默认支持 404、401、403、500、503 这几种错误的处理。用户可以自定义相应的错误处理，例如下面重新定义 404 页面：

	func page_not_found(rw http.ResponseWriter, r *http.Request){
		t,_:= template.New("beegoerrortemp").ParseFiles(beego.ViewsPath+"/404.html")
		data :=make(map[string]interface{})
		data["content"] = "page not found"
		t.Execute(rw, data)
	}
	
	func main() {
		beego.Errorhandler("404",page_not_found)
		beego.Router("/", &controllers.MainController{})
		beego.Run()
	}

我们可以通过自定义错误页面 `404.html` 来处理404错误。

Beego 更加人性化的还有一个设计就是支持用户自定义字符串错误类型处理函数，例如下面的代码，用户注册了一个数据库出错的处理页面：

	func dbError(rw http.ResponseWriter, r *http.Request){
		t,_:= template.New("beegoerrortemp").ParseFiles(beego.ViewsPath+"/dberror.html")
		data :=make(map[string]interface{})
		data["content"] = "database is now down"
		t.Execute(rw, data)
	}
	
	func main() {
		beego.Errorhandler("dbError",dbError)
		beego.Router("/", &controllers.MainController{})
		beego.Run()
	}

一旦在入口注册该错误处理代码，那么你可以在任何你的逻辑中遇到数据库错误调用 `this.Abort("dbError")` 来进行异常页面处理。


## response 处理

response 可能会有几种情况：

1. 模板输出

	上面模板介绍里面已经介绍，beego 会在执行完相应的 Controller 里面的对应的 Method 之后输出到模板。

2. 跳转

	上一节介绍的跳转就是我们经常用到的页面之间的跳转

3. 字符串输出

	有些时候我们只是想输出相应的一个字符串，那么我们可以通过如下的代码实现
	
		this.Ctx.WriteString("ok")
