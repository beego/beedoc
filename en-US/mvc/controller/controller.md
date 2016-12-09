---
name: Controller funcs
sort: 3
---

# Introduction to controller

> From version 1.6: `this.ServeJson()` is `this.ServeJSON()` and `this.TplNames` is `this.TplName`

Based on the design of Beego's controller, you just need to embed the `beego.Controller`:

	type xxxController struct {
	    beego.Controller
	}

`beego.Controller` implements interface `beego.ControllerInterface`.  `beego.ControllerInterface` defines these functions:

- Init(ct *context.Context, controllerName, actionName string, app interface{})

  This function will initialize Context, Controller name, template name, template variable container `Data`. `app` is the executing Controller's reflecttype. It can be used to execute subclass's methods.

- Prepare()

  You can use this function for extension, it will execute before the methods below. You can overwrite it to implement functions such as user validation.

- Get()

  This method will be executed if the HTTP request method is GET, and returns 403 by default. You can implement this method to handle GET requests by overwriting it in the struct of subclass.

- Post()

  This method will be executed if the HTTP request method is POST, and returns 403 by default. You can implement this method to handle POST requests by overwriting it in the struct of subclass.

- Delete()

  This method will be executed if the HTTP request method is DELETE, and returns 403 by default. You can implement this method to handle DELETE requests by overwriting it in the struct of subclass.

- Put()

  This method will be executed if the HTTP request method is PUT, and returns 403 by default. You can implement this method to handle PUT requests by overwriting it in the struct of subclass.

- Head()

  This method will be executed if the HTTP request method is HEAD, return 403 by default. You can implement this method to handle HEAD request by overwriting it in the struct of subclass.

- Patch()

  This method will be executed if the HTTP request method is PATCH, and returns 403 by default. You can implement this method to handle PATCH requests by overwriting it in the struct of subclass.

- Options()

  This method will be executed if the HTTP request method is OPTIONS, and returns 403 by default. You can implement this method to handle OPTIONS requests by overwriting it in the struct of subclass.

- Finish()

  This method will be executed after finishing the related HTTP method. It is empty by default. You can implement this method by overwriting it in the struct of subclass. It is used for database closing, data cleaning and so on.

- Render() error

  This method is used to render templates. It is only executed if `beego.AutoRender` is set to true.

By overwriting functions in struct, you can implement your own logic. For example:

```
type AddController struct {
    beego.Controller
}

func (this *AddController) Prepare() {

}

func (this *AddController) Get() {
    this.Data["content"] = "value"
    this.Layout = "admin/layout.html"
    this.TplName = "admin/add.tpl"
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
```

In the example above, it has implemented a RESTful structure by overwriting functions.

Now let's see a popular architecture: Implementing a baseController and some initializing methods. Other controllers then just inherit from it.

```
type NestPreparer interface {
        NestPrepare()
}

// baseRouter implements global settings for all other routers.
type baseRouter struct {
        beego.Controller
        i18n.Locale
        user    models.User
        isLogin bool
}
// Prepare implements Prepare method for baseRouter.
func (this *baseRouter) Prepare() {

        // page start time
        this.Data["PageStartTime"] = time.Now()

        // Setting properties.
        this.Data["AppDescription"] = utils.AppDescription
        this.Data["AppKeywords"] = utils.AppKeywords
        this.Data["AppName"] = utils.AppName
        this.Data["AppVer"] = utils.AppVer
        this.Data["AppUrl"] = utils.AppUrl
        this.Data["AppLogo"] = utils.AppLogo
        this.Data["AvatarURL"] = utils.AvatarURL
        this.Data["IsProMode"] = utils.IsProMode

        if app, ok := this.AppController.(NestPreparer); ok {
                app.NestPrepare()
        }
}
```

The above example defines a base class and initializes some variables. It will test if the executing Controller is an implementation of NestPreparer. If it is, then it calls the method of subclass. Let's see the implementation of `NestPreparer`:

```
type BaseAdminRouter struct {
    baseRouter
}

func (this *BaseAdminRouter) NestPrepare() {
    if this.CheckActiveRedirect() {
            return
    }

    // if user isn't admin, then logout user
    if !this.user.IsAdmin {
            models.LogoutUser(&this.Controller)

            // write flash message
            this.FlashWrite("NotPermit", "true")

            this.Redirect("/login", 302)
            return
    }

    // current in admin page
    this.Data["IsAdmin"] = true

    if app, ok := this.AppController.(ModelPreparer); ok {
            app.ModelPrepare()
            return
    }
}

func (this *BaseAdminRouter) Get(){
	this.TplName = "Get.tpl"
}

func (this *BaseAdminRouter) Post(){
	this.TplName = "Post.tpl"
}
```

Here is our logic: first execute `Prepare`. Go will search for methods in the struct by looking in the parent classes. While executing `BaseAdminRouter`, it checks whether there is a `Prepare` method. If not, it keeps searching `baseRouter`. If yes, it will execute the logic and `this.AppController` in `baseRouter` is the currently executing Controller `BaseAdminRouter`. Then it will execute `BaseAdminRouter.NestPrepare` method. Then it will start executing the related `GET` or `POST` method.


## Stop controller executing immediately

Sometime we want to stop the following execution logic of the request immediately and return the response. When we authenticate user in `Prepare` method, if the authentication failed, we return a response directly. You can use `StopRun()` to do it. 

```
type RController struct {
    beego.Controller
}

func (this *RController) Prepare() {
    this.Data["json"] = map[string]interface{}{"name": "astaxie"}
    this.ServeJSON()
    this.StopRun()
}
```

>>> If you call `StopRun`, the `Finish` method won't be run. If you need to free resources, please call `Finish` manually before call `StopRun`.


## Using PUT method in HTTP form

XHTML 1.x forms only support GET and POST. GET and POST are the only allowed values for the "method" attribute. But if you want to, you can simply do this:

Add a hidden input in the post form:

```
<form method="post" ...>
  <input type="hidden" name="_method" value="put" />
```

Then add a filter in Beego to check if the requests should be treated as a put request:

```go
var FilterMethod = func(ctx *context.Context) {
    if ctx.BeegoInput.Query("_method")!="" && ctx.BeegoInput.IsPost(){
          ctx.Request.Method = ctx.BeegoInput.Query("_method")
    }
}

beego.InsertFilter("*", beego.BeforeRouter, FilterMethod)
```
