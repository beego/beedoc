# Controller

Use `beego.controller` as anonymous in your controller struct to implement the interface in Beego:

	type xxxController struct {
		beego.Controller
	}

`beego.Controller` implemented`beego.ControllerInterface`, `beego.ControllerInterface` defined following methods:

- Init(ct *Context, cn string)

	Initialize context, controller's name, template's name, and container of template arguments

- Prepare()

	This is for expend usages, it executes before all the following methods. Users can overload this method for verification for example.

- Get()

	This method executes when client sends request as GET method, 403 as default status code. Users overload this method for customized handle process of GET method.

- Post()

	This method executes when client sends request as POST method, 403 as default status code. Users overload this method for customized handle process of POST method.

- Delete()

	This method executes when client sends request as DELETE method, 403 as default status code. Users overload this method for customized handle process of DELETE method.

- Put()

	This method executes when client sends request as PUT method, 403 as default status code. Users overload this method for customized handle process of PUT method.

- Head()

	This method executes when client sends request as HEAD method, 403 as default status code. Users overload this method for customized handle process of HEAD method.

- Patch()

	This method executes when client sends request as PATCH method, 403 as default status code. Users overload this method for customized handle process of PATCH method.

- Options()

	This method executes when client sends request as OPTIONS method, 403 as default status code. Users overload this method for customized handle process of OPTIONS method.

- Finish()

	This method executes after corresponding method finished, empty as default. User overload this method for more usages like close database, clean data, etc.

- Render() error

	This method is for rendering template, it executes automatically when you set beego.AutoRender to true.

Overload all methods for all customized logic processes, let's see an example:

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
