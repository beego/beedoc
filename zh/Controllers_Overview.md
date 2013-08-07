# 控制器（Controllers）

基于 Beego 的 Controller 设计，只需要匿名组合 `beego.Controller` 就可以了，如下所示：

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